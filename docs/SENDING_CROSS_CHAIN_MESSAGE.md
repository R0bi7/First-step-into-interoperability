# Sending cross-chain message

xCall encapsulates the underlying interoperability technology into a simple-to-use Smart Contract application programming interface (API).

Sending a cross-chain message should be as simple as calling the appropriate send message method of the pre-deployed xCall Smart Contract on the source chain and invoking the execute call method on the destination chain.

Purpose of this document is to:
- Present a [Simplified Flow Diagram](#simplified-send-message-flow-diagram)
- Provide an example implementation and invocation of the [sendCallMessage](#send-call-message-method) method
- Present the Smart Contract [Events](#events) output in the process
- Showcase an example implementation of [sending a message from the client side](#sending-message-from-client-side)
- Guide you to the [fee handling documentation](FEE_HANDLING.md)
- Direct you to the [error/failure handling scenario documentation](ERROR_HANDLING.md)


## Simplified Send Message Flow Diagram

![xCall flow diagram](imgs/xCall%20flow%20diagram.svg)

Send message flow:
1. The client invokes the [sendCallMessage](#send-call-message-method) method indirectly through Smart Contract A or directly to the xCall Smart Contract.
2. The [sendCallMessage](#send-call-message-method) method is invoked on the xCall Smart Contract, returning the request serial number and outputting the [CallMessageSent Event](#callmessagesent).
3. The [CallMessageSent Event](#callmessagesent) on the source chain is observed by the Relayer and relayed further to the destination chain's xCall Smart Contract, outputting the [CallMessage Event](#callmessage).
4. The [CallMessageSent Event](#callmessagesent) on the destination chain is observed by the client, which invokes the [executeCall](#execute-call-on-destination-chain) method in the xCall Smart Contract.
5. The invocation of the [executeCall](#execute-call-on-destination-chain) method results in the xCall Smart Contract invoking the [handleCallMessage](#handle-call-message-on-destination-chain) method of Smart Contract B.
6. Successful invocation of the [executeCall](#execute-call-on-destination-chain) method results in the xCall Smart Contract outputting the [CallExecutedEvent](#callexecuted-event).
7. If the input ``_rollback`` (in [sendCallMessage](#send-call-message-method)) was non-null, the [ResponseMessage Event](ERROR_HANDLING.md#responsemessage) is relayed back to the source chain.
8. If the execution failed and the input ``_rollback`` (in [sendCallMessage](#send-call-message-method)) was non-null, the failure flow described in [Error Handling](ERROR_HANDLING.md) is invoked.


## Send Call Message Method

Sending a call message requires your Smart Contract to call the **pre-deployed** xCall Smart Contract's `sendCallMessage` method.

More detailed explanation can be found [here](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-52.md#sendcallmessage).

### JVM
```java
/**
 * Sends a call message to the contract on the destination chain.
 *
 * @param _to The BTP address of the callee on the destination chain
 * @param _data The calldata specific to the target contract
 * @param _rollback (Optional) The data for restoring the caller state when an error occurred
 * @return The serial number of the request
 */
@Payable
@External
BigInteger sendCallMessage(String _to, byte[] _data, @Optional byte[] _rollback);
```

Which is part of [CallService.java](https://github.com/R0bi7/xCall-testing-JVM/blob/main/Commons/Interfaces/src/main/java/com/protokol7/interfaces/xcall/CallService.java)
interface.

You can find xCall Smart Contract implementation of ``CallService`` interface in [XCallProxy.java](https://github.com/R0bi7/xCall-testing-JVM/blob/main/Contracts/dapp-sample/src/main/java/foundation/icon/btp/xcall/sample/XCallProxy.java).

### EVM
```solidity
/**
   @notice Sends a call message to the contract on the destination chain.
   @param _to The BTP address of the callee on the destination chain
   @param _data The calldata specific to the target contract
   @param _rollback (Optional) The data for restoring the caller state when an error occurred
   @return The serial number of the request
 */
function sendCallMessage(
    string memory _to,
    bytes memory _data,
    bytes memory _rollback
) external payable returns (
    uint256
);
```


Which is part of [ICallService.sol](https://github.com/R0bi7/xCall-testing-EVM/blob/master/contracts/interfaces/xCall/ICallService.sol)
interface.

You can find xCall implementation of ``ICallService`` interface in [CallService.sol](https://github.com/icon-project/btp2-solidity/blob/276a7d7b004bcc918b6c4bf656439c335a3960b5/xcall/contracts/CallService.sol).


### Smart Contract Example Implementation

See below example implementations of JVM and EVM Smart Contracts able to send message.

We will explain how to handle errors and rollback in [Error Handling chapter](ERROR_HANDLING.md).

### JVM
```java
@Payable
@External
public void sendMessage(String _to, byte[] _data, @Optional byte[] _rollback) {
    if (_rollback != null) {
        // The code below is not actually necessary because the _rollback data is stored on the xCall side,
        // but in this example, it is needed for testing to compare the _rollback data later.
        var id = getNextId();
        Context.println("DAppProxy: store rollback data with id=" + id);
        RollbackData rbData = new RollbackData(id, _rollback);
        var ssn = _sendCallMessage(Context.getValue(), _to, _data, rbData.toBytes());
        rbData.setSvcSn(ssn);
        rollbacks.set(id, rbData);
    } else {
        // This is for one-way message
        _sendCallMessage(Context.getValue(), _to, _data, null);
    }
}
```

[Source code](https://github.com/R0bi7/xCall-testing-JVM/blob/main/Contracts/dapp-sample/src/main/java/foundation/icon/btp/xcall/sample/DAppProxySample.java).

### EVM
```solidity
function sendMessage(
    string calldata _to,
    bytes calldata _data,
    bytes calldata _rollback
) external payable {
    if (_rollback.length > 0) {
        uint256 id = ++lastId;
        bytes memory encodedRd = abi.encode(id, _rollback);
        uint256 sn = ICallService(callSvc).sendCallMessage{value:msg.value}(
            _to,
            _data,
            encodedRd
        );
        rollbacks[id] = RollbackData(id, _rollback, sn);
    } else {
        ICallService(callSvc).sendCallMessage{value:msg.value} (
            _to,
            _data,
            _rollback
        );
    }
}
```

[Source code](https://github.com/R0bi7/xCall-testing-EVM/blob/master/contracts/HelloWorld.sol).

## Events

Events are an important part of xCall messaging flow because they provide us with relevant information about the execution flow and state.

They are an integral part of sending cross-chain message and should be handled appropriately. 

### CallMessageSent

When xCall invokes ``sendMessage`` in BMC, it returns nsn (network serial number) that identifies each BTP message from the source chain.
The following event is emitted after xCall receives the nsn from BMC, and can be used for BTP message tracking purpose.

```java
/**
 * Notifies that the requested call message has been sent.
 *
 * @param _from The chain-specific address of the caller
 * @param _to The BTP address of the callee on the destination chain
 * @param _sn The serial number of the request
 * @param _nsn The network serial number of the BTP message
 */
@EventLog(indexed=3)
void CallMessageSent(Address _from, String _to, BigInteger _sn, BigInteger _nsn);
```

### CallMessage

When the xCall on the destination chain receives the call request through BMC, it emits the following event for notifying the user.

```java
/**
 * Notifies the user that a new call message has arrived.
 *
 * @param _from The BTP address of the caller on the source chain
 * @param _to A string representation of the callee address
 * @param _sn The serial number of the request from the source
 * @param _reqId The request id of the destination chain
 * @param _data The calldata
 */
@EventLog(indexed=3)
void CallMessage(String _from, String _to, BigInteger _sn, BigInteger _reqId, byte[] _data);
```

To minimize the gas cost, the calldata payload delivered from the source chain are exported to event ``_data`` field, instead of storing it in the state db. The _data payload should be repopulated by the user
(or client) when calling the following ``executeCall`` method. Then ``xCall`` compares it with the saved hash value
to validate its integrity.

## Execute Call On The Destination Chain
After ```CallMessage``` Event is observed by user (client) he invokes the following method on ``xCall`` with the given ``_reqId`` and ``_data``.

```java
/**
 * Executes the requested call message.
 *
 * @param _reqId The request id
 * @param _data The calldata
 */
@External
void executeCall(BigInteger _reqId, byte[] _data);
```

## Handle Call Message On The Destination Chain

When the user calls ``executeCall`` method, the ``xCall`` invokes the following predefined method
in the target dApp with the calldata associated in ``_reqId``.

**Note**: if execution fails on the destination chain, `handleCallMessage` may also need to be defined in the source chain Smart Contract A
in order to handle the [executeRollback](ERROR_HANDLING.md#executerollback) invocation properly!

````java
/**
 * Handles the call message received from the source chain.
 * Only called from the Call Message Service.
 *
 * @param _from The BTP address of the caller on the source chain
 * @param _data The calldata delivered from the caller
 */
@External
void handleCallMessage(String _from, byte[] _data);
````


If the call request was a one-way message and dApp on the destination chain needs to send back the result (or error),
it may call the same method interface (i.e. ``sendCallMessage``) to send the result message to the caller.
Then the user on the source chain would be notified via ``CallMessage event``, and call ``executeCall``,
then DApp on the source chain may process the result in the ``handleCallMessage`` method.

### CallExecuted Event

To notify the execution result of dApp's ``handleCallMessage`` method, the following event is emitted after its execution.

```java
/**
 * Notifies that the call message has been executed.
 *
 * @param _reqId The request id for the call message
 * @param _code The execution result code
 *              (0: Success, -1: Unknown generic failure, >=1: User defined error code)
 * @param _msg The result message if any
 */
@EventLog(indexed=1)
void CallExecuted(BigInteger _reqId, int _code, String _msg);
```

## Sending Message From The Client Side

In order to send message from source to destination chain off-chain call using user wallet must be made.

What if I told you that sending xCall message can be as short as few lines of typescript code?

Example:
```typescript
const xCallService = new XCallService();
const abiCoder = new ethers.utils.AbiCoder();

// EXAMPLE: icon to bsc no rollback
let message = abiCoder.encode([ "uint8", "bytes" ], [ 0, ethers.utils.toUtf8Bytes("Hello World") ]);
await xCallService.sendCallMessage(Network.icon, Network.bsc, message, true , true);
```
[Source code](https://github.com/R0bi7/xCall-testing-dApp/blob/master/src/dApp.ts).

Checkout [XCallService](https://github.com/R0bi7/xCall-testing-dApp/blob/master/src/services/XCallService.ts) to see how we implemented typescript service
being able to send message from EVM/JVM to EMV/JVM in a simple manner.

[xCall-testing-dApp](https://github.com/R0bi7/xCall-testing-dApp) repository showcases how simplified xCall messaging can be.

**Note**: proper configuration and deployment of before mentioned Smart Contracts is needed.

### Sending Varius Types Of Messages

Message being sent takes form in well known `bytes` thus Client has various options on how to encode and decode messages on sending and receiving end.

You should use more sophisticated encoding such as [RLP](https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp/) in production.

**Note**: messages in examples are encoded in `uint8` and `bytes` format, where `uint8` represents message type used on receiving end correctly parse different types
and `bytes` which represent encoded string.

#### Examples

```typescript
// EXAMPLE: icon to bsc no rollback
let message = abiCoder.encode([ "uint8", "bytes" ], [ 0, ethers.utils.toUtf8Bytes("Hello World") ]);
await xCallService.sendCallMessage(Network.icon, Network.bsc, message);
```
[Source code](https://github.com/R0bi7/xCall-testing-dApp/blob/master/src/dApp.ts).

```typescript
// EXAMPLE: icon to bsc with rollback
let message = abiCoder.encode([ "uint8", "bytes" ], [ 0, ethers.utils.toUtf8Bytes("revertMessage") ]);
await xCallService.sendCallMessage(Network.icon, Network.bsc, message, true , true);
```
[Source code](https://github.com/R0bi7/xCall-testing-dApp/blob/master/src/dApp.ts).

```typescript
// EXAMPLE: icon to bsc with concatenation on receiving end
let message = abiCoder.encode([ "uint8", "bytes" ], [ 1, ethers.utils.toUtf8Bytes("hello w") ]);
await xCallService.sendCallMessage(Network.icon, Network.bsc, message);
```
[Source code](https://github.com/R0bi7/xCall-testing-dApp/blob/master/src/dApp.ts).

```typescript
// EXAMPLE: icon to bsc with multiple messages in single message
const stringArray = ["hello world 1", "hello world 2", "hello world 3"];
const encodedArray = ethers.utils.defaultAbiCoder.encode(["string[]"], [stringArray]);
let message = abiCoder.encode([ "uint8", "bytes" ], [ 2, encodedArray ]);
await xCallService.sendCallMessage(Network.icon, Network.bsc, message);
```
[Source code](https://github.com/R0bi7/xCall-testing-dApp/blob/master/src/dApp.ts).

## Fee Handling

Elaboration on Fee handling can be found in [Fee Handling](FEE_HANDLING.md) document.

## Error Handling

Elaboration on Error handling can be found in [Error Handling](ERROR_HANDLING.md) document.

## Relevant Resources

- [xCall-testing-dApp](https://github.com/R0bi7/xCall-testing-dApp)
- [Pre-deployed xCall & BTP Smart Contracts](https://docs.icon.community/cross-chain-communication/blockchain-transmission-protocol-btp)
- [xCall Standard documentation](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-52.md)