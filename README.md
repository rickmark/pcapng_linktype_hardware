# pcapng_linktype_spi

A proposal for a SPI linktype for `libpcap`

## Reason

Today there are several network protocols covered by the standard pcapng format.  This is because the pcapng format was intended to serve as a generic encapsulation of interfaces, and the binary data passed to and from enpoints on that interface.  This led to the generic adoption of `pcapng` to begin to represent other protocols such as USB.  The proposal is to create a specified and standardized method of encapsulating other hardware protocols, the first of which being commonly used in many embedded systems today as SPI.  On top of SPI which is a generic serial link, a common usage is the JEDEC flash memory standard.  Because SPI lacks all of the required descriptors, the specification of this "next layer protocol" must often be explicitly specified.

## Decision Points

### Multiple Slave Devices

SPI is a master to slave protocol, with the capability of using multiple CS (chip-selects) to specify the device to which the master speaks.  This implies that an "interface" in SPI packet capture parlence should be 1:1 with a master, and that there should be a reasonable number of devices specified by integer as to which device a transfer occurs.

An alternative would be to specify one interface per M/S pair.  This has the addtitional advantage of allowing the embedding of pairwise data in the interface record, but would loose critical data about how the links inter-relate.

A final design possability is the one interface per master, embedding N slave details in the interface.

This might best be represented by a new block type of "BLOCK_LINK_TOPOLOGY" - it would include some amount of standard data about if the link is 1:1, 1:many or many:many, if the link is up or down, if the link supports resets or state changes, if the link is full/half duplex or if that can be changed, if the link has the ability to transition speeds or encodings.  The block would also encapsulate a namespace of options specific to the LINKTYPE for which it is related.  For SPI this is a great place to put any "next layer protocol" data like JEDEC as that is M/S link specific.

It seems reasonable to make the assumption that all master slave relationships are fixed for the span of a transfer.

### Handling Link Reset and Speed Changes

The RESET line allows a M/S to transition back through initial state.  This as well as SPI => Dual / Quad SPI changes should be logically captured as link level events, but are not themselves particularly interesting as the frames captured (one can almost think of RESET as a side channel to the actual transfers and lacks a way to repressent as an octet stream).

This seems to be an unhandled case of pcapng itself.  A proposal would be to create a new block type for `BLOCK_LINK_CHANGE` that would allow resets, speed changes etc to be sideband data to the packet frames.  The SPI format would plumb through some relevent data (e.g. SPI/Dual/Quad) in every EPB as to free the readers from having to maintain a state machine to process these events.

## Transfer Frames

Each transfer frame should include direction/target, SPI/DualSPI/QuadSPI, timing, clock rate (average/skew), and raw data.  They should also include a monotonic increasing value indicating number of resets (so that each stream of transfers can be correlated.

