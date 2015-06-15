# Gigabyte GA-Z77X Graphics Patches

DSDT patches for various graphics cards on OS X.

### Using These Patches
These patches can be applied to a cleanly-extracted native DSDT from the Gigabyte GA-Z77X-UD5H motherboard using an editor such as [MaciASL](http://maciasl.sourceforge.net). Add `https://raw.githubusercontent.com/theracermaster/Gigabyte-GA-Z77X-Graphics-DSDT-Patch/master` as a repo in MaciASL > Preferences > Sources.

### Special Note
If you want to use both Intel HD Graphics 4000 and a discrete card, you must set IGFX as Display Init (default GPU) in the BIOS, and connect your primary monitor to the IGPU. Apply the Intel HD Graphics 4000 patch first, then the patch for your discrete card (make sure you edit all instances of `onboard-1` to `onboard-2` in the discrete card's patch).

### Intel Integrated Graphics
If you used one of the GA-Z77X post installation scripts in one of the repos, your DSDT was automatically patched with the right DSDT patch for Intel integrated graphics. If you want to change the patch used (e.g. you are using a discrete GPU, but also want to use the integrated GPU for other displays), or manually applied the DSDT patches, you can apply one of the patches located here.
#### AirPlay Mirroring
The AirPlay mirroring DSDT patch for Intel HD Graphics 4000 uses an ig-platform-id value that disables all video outputs of the integrated GPU, but keeps it active for other activites such as QuickSync. This is the same method real iMacs use to enable AirPlay mirroring while "disabling" (it is simply hidden in System Information and has all outputs disabled) the integrated GPU. If you want to use both Intel HD Graphics 4000 and a discrete card (not for AirPlay mirrorring), you must set IGFX as Display Init (default GPU) in the BIOS, and connect your primary monitor to the IGPU. Apply the Intel HD Graphics 4000 patch first, then the patch for your discrete card (make sure you edit all instances of `onboard-1` to `onboard-2` in the discrete card's patch).

### AMD Discrete Graphics
#### Customizing These Patches For Your Card
The 7970 patch in this repo is specifically for the Sapphire Radeon HD 7970 Dual-X OC with Boost using the Hamachi framebuffer. If you want to make some minor cosmetic adjustments, you can change the ROM number (this can usually be found in the VBIOS of your card). You will have to change the connector types as well (this info can be found in a VBIOS dump using either `redsock_bios_decoder` and/or `radeon_bios_decode`). You may also have to change the framebuffer (some cards may need a patched framebuffer - see [RampageDev's guide](http://www.rampagedev.com/?page_id=82&page=10) for more info. [NOTE: Use `radeon_bios_decode` to find OS X sense IDs.])

#### Editing AMD Framebuffer Personalities
The Sapphire Radeon HD 7970 Dual-X OC with Boost has 2x DVI, 1x HDMI, and 1x DP. None of the stock 7000 personalities work with this card, so we need to patch the framebuffer (in this case, Hamachi).

We must patch the AMD7000Controller kext and replace the Hamachi framebuffer with our modified one. This can be done in multiple ways.

- Clover auto-patching (recommended, applied at every boot)

  Insert this in your config.plist, under KextsToPatch.
  
  ```plist
  <dict>
    <key>Comment</key>
    <string>Hamachi Framebuffer Patch</string>
    <key>Find</key>
    <data>AAQAAAQDAAAAAQEBEgQFAQAEAAAEAwAAAAECASIFBAIEAAAAFAIAAAABAwAAAAMGAAgAAAQCAAAAAQQAEQIBBA==</data>
    <key>Name</key>
    <string>AMD7000Controller</string>
    <key>Replace</key>
    <data>AAQAAAQDAAAAAQAAEgQBAQAIAAAEAgAAAAEAACIFAgQAAgAABAIAAAABAAARAgMDBAAAABQCAAAAAQAAEAAEBg==</data>
  </dict>
  ```

- Manually patching the kext (needs to be applied after every OS update)

  Replace the stock framebuffer with our patched one:

  `sudo perl -pi -e 's|\x00\x04\x00\x00\x04\x03\x00\x00\x00\x01\x01\x01\x12\x04\x05\x01\x00\x04\x00\x00\x04\x03\x00\x00\x00\x01\x02\x01\x22\x05\x04\x02\x04\x00\x00\x00\x14\x02\x00\x00\x00\x01\x03\x00\x00\x00\x03\x06\x00\x08\x00\x00\x04\x02\x00\x00\x00\x01\x04\x00\x11\x02\x01\x04|\x00\x04\x00\x00\x04\x03\x00\x00\x00\x01\x00\x00\x12\x04\x01\x01\x00\x08\x00\x00\x04\x02\x00\x00\x00\x01\x00\x00\x22\x05\x02\x04\x00\x02\x00\x00\x04\x02\x00\x00\x00\x01\x00\x00\x11\x02\x03\x03\x04\x00\x00\x00\x14\x02\x00\x00\x00\x01\x00\x00\x10\x00\x04\x06|g' /System/Library/Extensions/AMD7000Controller.kext/Contents/MacOS/AMD7000Controller`
  
  Rebuild kext caches:
  
  `sudo touch /System/Library/Extensions`

  And then reboot.

### NVIDIA Discrete Graphics
In most cases, you will not need to patch the DSDT for NVIDIA GPUs if you are using [the NVIDIA Web Drivers](http://www.insanelymac.com/forum/topic/301416-nvidia-web-driver-updates-for-yosemite/). Make sure that you use the correct version for not just your version of OS X, but your specific build of OS X. Note that if you are using a Fermi GPU, you will most likely encounter channel exceptions in recent versions of OS X, leading to the dreaded Fermi freeze. To fix this, you can you use [FreezeFix](http://www.insanelymac.com/forum/topic/291300-fermi-freeze-investigation/) by dan542. Make sure you add it as a login item in order to prevent channel exceptions and freezing.
