# brother-mfc-j6955dw

Arch Linux package for the Brother MFC-J6955DW CUPS printer driver, built from
Brother's official `mfcj6955dwpdrv-3.5.0-1.i386.deb` and patched so it works
properly with current Arch (CUPS 2.4, libcupsfilters 2).

## What the patch fixes

- **Print settings stuck on the default profile.** Brother's filter only read
  its own option names, so the standard settings sent by GTK, Qt, LibreOffice
  and browser print dialogs (colour mode, quality, two-sided) were ignored.
- **Paper source, media type and borderless sizes not offered in dialogs.**
  The PPD now uses the standard `InputSlot`/`MediaType` keywords and
  `*.Fullbleed` borderless size names, so CUPS can map them to IPP. The
  wrapper also accepts stand-alone `media-source=`/`media-type=` options and
  `media=a,b,c` lists, which cupsd does not translate on its own.
- **Draft/Normal/High quality in dialogs.** `APPrinterPreset` entries map IPP
  `print-color-mode` × `print-quality` to Brother's `BRMonoColor`/`BRResolution`.
- **Upside-down back pages in long-edge duplex.** libcupsfilters' `pdftopdf`
  already rotates back pages for `sides=two-sided-long-edge`, and Brother's
  filter rotated them again.
- **Arch-specific issues.** `which` is not in Arch `base` (the filter needed it
  to find ghostscript); the LPD/printcap and USB-queue install steps are
  dropped; only the x86_64 binaries are installed; error spam from options this
  model does not support is removed; temp/lock files use CUPS' `TMPDIR`.

All of the above was verified with test prints on a real MFC-J6955DW.

## Build and install

```
makepkg -si
```

Add the queue (network printer; replace the IP):

```
sudo lpadmin -p MFC-J6955DW -E -v lpd://192.168.1.52/binary_p1 \
  -P /usr/share/ppd/Brother/brother_mfcj6955dw_printer_en.ppd
```

Set defaults with the standard IPP options, e.g.:

```
lpadmin -p MFC-J6955DW -o print-color-mode-default=monochrome -o print-quality-default=5
```

After an upgrade that changes the PPD, refresh the queue's copy:

```
sudo lpadmin -p MFC-J6955DW -P /usr/share/ppd/Brother/brother_mfcj6955dw_printer_en.ppd
```

## Driverless alternative

The printer also supports IPP Everywhere, which works without this package and
will keep working once CUPS 3 drops PPD drivers:

```
sudo lpadmin -p MFC-J6955DW-Driverless -E -v ipp://192.168.1.52/ipp/print -m everywhere
```

This driver is still needed for Secure Print (PIN) and Japanese paper sizes.

## Licence

Brother's driver is distributed under Brother's licence (see the LICENSE file
installed to `/usr/share/licenses/brother-mfc-j6955dw/`), which permits
redistribution and modification. The CUPS wrapper scripts are GPL-2.0-or-later.
