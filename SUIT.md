# Software Update for IoT (SUIT)

The IETF [SUIT Working
Group](https://datatracker.ietf.org/wg/suit/about/) seeks to defining
a software update solution for IoT devices.  It's primary output is a
document known as [the
manifest](https://datatracker.ietf.org/doc/draft-ietf-suit-manifest/).

There is a lot of overlap between what MCUboot accomplishes and what
the SUIT WG seeks to define.  There are also some significant
differences that must be understood and addressed in order to
determine the applicability of the SUIT manifest to MCUboot.

## The existing manifest

Please see [the design
documentation](https://mcuboot.com/mcuboot/design.html) for details
about the MCUboot structures and data.  This document will be
primarily concerned with the `image_header`, the image itself, and the
TLV payload that follows the image.

The data contained in the image header, and the TLV(s) that
follow the image represent the same information that would be encoded
in a SUIT manifest.

The image header is a small structure placed at the beginning of the
partition.  It is designed to be easy to access, easy to determine
some measure of validity, and contains mostly information about the
sizes of the remaining images.  In addition, it contains a load
address field which is used by load-to-ram solutions to determine the
address the image should be loaded to.  It also contains a flags
field, which mostly determines whether the image should be loaded to
ram, and for a special case of a split image app.

The header is of a variable size, although, in practice, the size is
determined by the particular target, and is needed to allow the
beginning of the image itself to fall on a particular alignment.  The
largest value this can take on Cortex-M devices is 1024 bytes.

After this padded header is the executable image itself.  The format
of this image is defined by the architecture running, and is often of
a similar format to how images that are directly run are formatted.
In the case of Cortex-M, the first two `uint32_t` values are the
initial stack pointer, and the reset address.

Immediately following the image are one or two TLV-encoded manifest
blocks (just called the TLV in the code).  The first, or protected,
TLV is included in the image hash and is used to hold dependency
information.  The second contains information about a digital
signature of the previous image.  What is signed consists of the
header, the image, and the protected TLV.

## The SUIT manifest

At a conceptual level, the SUIT manifest captures all of the
information contained in the image header and TLVs.  It is also
capable of capturing more information than this (for example
information on how to download images).  There are also differences in
how dependencies are managed.

### The single manifest

The first notable difference that can be noticed is that the SUIT
manifest holds in a single structure the information that is split
between the image header and the TLVs.  This gives us our first design
choice:

1.  Should we keep the image header, and just replace the TLV with the
    manifest.  This will make some information redundant (along with
    the question of what to do if it is incorrect).  However, this
    will also possibly require fewer code changes, as some of the code
    expects to be able to look at the image header to quickly read
    values such as length.  Not doing it this way will require
    additional abstraction of the code.

2.  The reference sample bootloader code places the SUIT manifest at
    the beginning of the flash slot, with a 1k padding which is
    followed by the image itself.  Although this will require
    additional changes to the MCUboot code, it would likely make a
    SUIT bootloader image more compatible with other tools developed
    to support SUIT.

In either case, the data stored at the end of the image (the trailer)
can be kept the same, as this is considered beyond the scope of SUIT.

### Declarative vs procedural

The MCUboot native manifest is stored in what is known as a
declarative format.  All of the fields and TLV entries make an
assertion of something that is to be true about the image.  It is
entirely up to the bootloader itself to determine what to do with this
information, and to make decisions as to how the image should be
verified, when upgrades should happen, and how the image should be
run.

Earlier drafts of SUIT also used this approach.  However, the latest
drafts use a procedural format.  After a small amount of declarative
information (the signatures are declarative), the body of the manifest
consists of essentially a byte code describing what the bootloader
should do.  The main advantage of this is that it makes the manifest
processing code much smaller.

However, it does change quite a bit the structure of the code.
Instead of code within MCUboot implementing the logic of whether to
upgrade, and run an image, that decision would be directed by
interpreting the manifest.  Given that the bootloader isn't
particularly complex, this change isn't significant, but it does
change fairly fundamentally how MCUboot would work.

### Dependency differences

SUIT is flexible enough to allow multiple ways of handling
dependencies.  It would be possible to treat them as a single image,
and have a single manifest describe two images.

It is also possible to have manifests that depend on other manifests.
This logic would be able to implement a dependency check, however, it
only supports depending on exact versions of the other images.  It was
stated that the intent, if an image is not to be updated, is that a
new manifest could be generated for the image, and the upgrade process
could detect an unchanged image, and avoid transmitting it.

## Reference code

The IETF working group has produced a body of reference code for
processing SUIT manifests within the context of a bootloader.  The
code seems well written, and is fairly small.

At the "Security the IoT Hackathon" in February, 2020, we worked
toward a prototype of using this code.  We accomplished modifying
imgtool to include the SUIT manifest generation code, currently
appending the manifest where the TLV would be.  An alternative
bootloader was able to use the manifest code to extract the sequence
number.  Additional work will need to be done to incorporate this more
seamlessly with the rest of MCUboot.
