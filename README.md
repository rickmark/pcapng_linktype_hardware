# pcapng_linktype_spi

A proposal for a SPI linktype for `libpcap`

## Reason

Today there are several network protocols covered by the standard pcapng format.  This is because the pcapng format was intended to serve as a generic encapsulation of interfaces, and the binary data passed to and from enpoints on that interface.  This led to the generic adoption of `pcapng` to begin to represent other protocols such as USB.  The proposal is to create a specified and standardized method of encapsulating other hardware protocols, the first of which being commonly used in many embedded systems today as SPI.  On top of SPI which is a generic serial link, a common usage is the JEDEC flash memory standard.  Because SPI lacks all of the required descriptors, the specification of this "next layer protocol" must often be explicitly specified.

## Decision Points

SPI is a master to slave protocol, with the capability of using multiple CS (chip-selects) to specify the device to which the master speaks.  This implies that an "interface" in SPI packet capture parlence should be 1:1 with a master, and that there should be a reasonable number of devices specified by integer as to which device a transfer occurs.

An alternative would be to specify one interface per M/S pair.  This has the addtitional advantage of allowing the embedding of pairwise data in the interface record, but would loose critical data about how the links inter-relate.

A final design possability is the one interface per master, embedding N slave details in the interface.

## Transfer Frames

Each transfer frame should include direction/target, SPI/DualSPI/QuadSPI, timing, clock rate (average/skew), and raw data.  They should also include a monotonic increasing value indicating number of resets (so that each stream of transfers can be correlated.
