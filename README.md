# Raspberrypi-kodi-service
Systemd service unit to run kodi-standalone on RPis with a minimal frame buffer footprint. Saves approx 7 MB of GPU memory + extra SDRAM bandwidth.

## Details
By default, booting to 1080p will allocate approx 7 MB of video memory to the frame buffer which is wasted when running Kodi. Dropping the size of the frame buffer down to a small setting reduces the 7 MB to nearly 0.

Allocated memory can be viewed using the `vcdbg` binary provided by the raspberrypi-firmware package.  Example frame buffer booting into the default 1080p resolution:
```
# vcdbg reloc | grep ARM
[   3] 0x3e512f80: used 6.9M (refcount 1 lock count 8, size  7237632, align 4096, data 0x3e513000, d1rual) 'ARM FB'
```

Example using this service:
```
# vcdbg reloc | grep ARM                  
[   3] 0x3ebf9b80: used 5.0K (refcount 1 lock count 8, size     1024, align 4096, data 0x3ebfa000, d1rual) 'ARM FB'
```

## Dependencies
* fbset (http://users.telenet.be/geertu/Linux/fbdev)
* kodi

__The service expects a user named "kodi" belonging to the "kodi" group to be present.__

## Installation
* Copy `kodi-framebuffer` into `/etc/conf.d/`
* Copy `kodi.service` into `/etc/systemd/system/`
* Refresh systemd via `systemd daemon-reload`

Optionally edit the environment file, `/etc/conf.d/kodi-framebuffer`, to change the target restore frame buffer mode when the service is stopped.  The default option is the 1080p setup.

## Acknowledgment
Inspiration for this work is from to the developers of LibreELEC project. 
* Disabling the frame buffer upon launching (although this alone does not free-up the reported memory): https://github.com/LibreELEC/LibreELEC.tv/blob/master/packages/graphics/bcm2835-driver/system.d/unbind-console.service
* Shrinking the frame buffer (hard-coded in their case): https://github.com/LibreELEC/LibreELEC.tv/blob/master/packages/mediacenter/kodi/patches/kodi-999.99.framebuffer-reset.patch
