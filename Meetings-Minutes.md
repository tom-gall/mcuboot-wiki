# Meeting notes

## 2020-02-25

### Agenda

- Report on IETF Hackathon (links)
- Statuses
- Cypress questions about upgrading mbed TLS

### Attendees

- David Brown
- nvlsianpu
- David Vincze
- Fabio
- Øyvind Rønningstat
- rnok

### Notes

- #205: dual-slot (non swap) discussed, and seems to be desired
- #668: Does Zephyr need a mechanism to shut down hardware.
- SUIT hackathon overview notes are on the wiki [[SUIT]]
  Unclear how much demand there will be for this feature.  SUIT does
  give a few benefits, mostly beyond the bootloader, but it does allow
  an image to be targeted toward a specific device.
- Pending pulls
  - Discussed #660, CBOR update for serial protocol.  We decided to
    keep both the submodule, as well as committing local copies of the
    two files needed to build.  This allows builds from Zephyr without
    needing to add additional modules to West.
- Question of policy on Zephyr versions
  - TODO: Need to make this explicit in the documetnation.  Policy:
    - MCUboot '''master''' always works with Zephyr '''master'''
    - An MCUboot release should be documented and work with a
      particular Zephyr version.
    - Other desired combinations should be fixed in the Zephyr fork of
      MCUboot.

# Older

Home page for collect thought and summaries from meetings.

5.10.2018, 14:00 UTC [[Multiple image support]]
