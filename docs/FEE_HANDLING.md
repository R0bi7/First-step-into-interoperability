# Fees Handling

If a user wants to make a call from ICON to Target Network 1 (T1), he needs to pay X ICX, and for Target Network 2 (T2),
he needs to pay Y ICX. That is, the fees depend on the destination network address.

The fees are divided into two types, one is for relays and the other is for protocol itself. For example,
for a destination network T1, the fees could be relayFee = 0.25 ICX and protocolFee = 0.01 ICX. And relayFee goes to relays,
protocolFee goes to BTP protocol (eventually, to the Fee Handler). In this document, we don't address how to deal with
these accrued fees for distribution, but just define operational parts like how to get the proper fee amount before
sending the call request, etc.

Here are getter and setter methods for the proper fees handling in ``xCall``. DApps that want to make a call to
``sendCallMessage``, should query the total fee amount for the destination network via ``getFee`` interface,
and then enclose the appropriate fees in the method call. Note that the protocol fee amount can be get/set via ``xCall``,
but the relay fee would be obtained from BMC which manages BTP network connections.

```java
/**
 * Gets the fee for delivering a message to the _net.
 * If the sender is going to provide rollback data, the _rollback param should set as true.
 * The returned fee is the sum of the protocol fee and the relay fee.
 *
 * @param _net The network address
 * @param _rollback Indicates whether it provides rollback data
 * @return the sum of the protocol fee and the relay fee
 */
@External(readonly=true)
BigInteger getFee(String _net, boolean _rollback);

/**
 * Sets the protocol fee amount.
 *
 * @param _value The protocol fee amount in loop
 * @implNote Only the admin wallet can invoke this.
 */
@External
void setProtocolFee(BigInteger _value);

/**
 * Gets the current protocol fee amount.
 *
 * @return The protocol fee amount in loop
 */
@External(readonly=true)
BigInteger getProtocolFee();

/**
 * Sets the address of Fee Handler.
 * If _addr is null (default), it accrues protocol fees.
 * If _addr is a valid address, it transfers accrued fees to the address and
 * will also transfer the receiving fees hereafter.
 *
 * @param _addr The address of Fee Handler
 * @implNote Only the admin wallet can invoke this.
 */
@External
void setProtocolFeeHandler(@Optional Address _addr);

/**
 * Gets the current protocol fee handler address.
 *
 * @return The protocol fee handler address
 */
@External(readonly=true)
Address getProtocolFeeHandler();
```