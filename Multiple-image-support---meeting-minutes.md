https://github.com/runtimeco/mcuboot/pull/317 of design proposal for supporting multiple image update in MCUBoot.

ARM decelerated that they will going to work on the implementation in Q4 this year

Summary meeting take place 5.10.2018, 14:00 UTC,
Original agenda by Tamas Ban (ARM)
---------------------------------------------------------------------------------------------
The plan is to discuss the open questions around the design proposal:
- whether need for auto revert
- terminology: secure_slot_0 vs. image_0_primary_slot
- how to describe dependencies: extend current TLV format vs. SUIT
- check boot flow with multiple images
Everybody: Please let us know if you interested in! If proposed time does not fit many of you then come up with alternatives.
--------------------------------------------------------------------------------------------

Discuses agenda
* Introductions
* Overview of topic: upstream multi-image support
* auto-revert needed?
    * Start with overwrite only? <-
    * Overwrite only mode will be Implemented first
        * Rollback support might be contributed as 2nd step (not decided when and who will do it)
* how to describe dependencies
    * lock stepped with some semantic version level compatibility
    * explicit dependencies, with semantic constraints. likely only needed in one direction (app depends on version of secure image) <-
        * David Brown suggest to add dependencies only to Non-secure image, seams it solve the requirements for now.
* extend TLV vs SUIT
    * Separate manifests
    * Wait on SUIT, do our own thing now
        * Implement TLV extension for now
    * Keep single signature scheme
        * Complex signature
* ~~check boot flow with multiple images~~ – not discussed as resigned from rollback implementation for this steep
* terminology
    * current slot 0 current, slot 1 upgrade
    * perhaps “primary” and “secondary” instead of secure non-secure
    * Getting away from specific words.
        * Suggested to change terminology right now, in separate PR
* simulator
    * Shared work, different parties may work on simulator
    * e.g. one team pushes C code change, and others may write the simulator code
        * ARM rather need help with the RUST simulator.
* qemu
    * flash device in qemu
    * or a simulated driver in Zephyr <-
        * possible to use this for some testing
* backend
    * Currently Zephyr, MyNewt
    * Do we add CMSIS
