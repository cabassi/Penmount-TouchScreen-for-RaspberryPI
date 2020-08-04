# Touchscreen Calibration in Debian9

Thanks to KurtJacobson who wrote this gist (https://gist.github.com/KurtJacobson/37288a0300a9c1b3e859c8dcff403300)


Unfortunately [`xinput-calibrator`][1] does not work at all for calibrating a 
touchscreen in Debian9. This is apparently because X server now uses libinput 
to handle input devices instead of evdev. I spent huge amount of trying to
fiddling with `xinput-calibrator` and `99-calibration.conf` files until I finely
found this [issue][2] on GitHub that gave me some hints as how to proceed. This
is mostly for my own reference, but I hope it might also help others in the same
situation.


### Install `xinput`

This not not seem to be installed by defaults on Debian9

`$ sudo apt-get install xinput`


### Determine the screen size

You probably already know this, but if you have multiple screens they might be
see as one big screen. So to determine the total size run

`$ xrandr`

This will print out a good bit of information, but what you are interested in is
the `current` vales in the first line, which will look something like this:

`Screen 0: minimum 320 x 200, current 1440 x 900, maximum 8192 x 8192`


### Determine the name of the touch device

Next step it to find the touch device's name

`$ xinput list`

Look for the touch device in the `Virtual core pointer` section. In my case the
device name is `Elo TouchSystems 2700 IntelliTouch(r)`.

```
⎡ Virtual core pointer                          id=2    [master pointer  (3)]
⎜   ↳ Virtual core XTEST pointer                id=4    [slave  pointer  (2)]
⎜   ↳ Elo TouchSystems 2700 IntelliTouch(r)     id=12   [slave  pointer  (2)]
⎜   ↳ 2.4G Mouse                                id=10   [slave  pointer  (2)]
⎣ Virtual core keyboard                         id=3    [master keyboard (2)]
    ↳ Virtual core XTEST keyboard               id=5    [slave  keyboard (3)]
...
```

Now run 

`$ xinput list-props <device name>`

were `<device-name>` is the name you determined in the last step.

Confirm that the list of properties includes near the top a property called 
`Coordinate Transformation Matrix`. If it does not, you probably have the wrong 
device name.

### Set the `Coordinate Transformation Matrix`

The screen is calibrated using a `Coordinate Transformation Matrix`, which
defaults to the identity matrix.

[1 0 0]  
[0 1 0]  
[0 0 1]  

for convenient entry, the matrix is flatted into a single line like this

`1 0 0 0 1 0 0 0 1`

were the valuse seem to map like this

`[hscale] [vskew] [hoffset] [hskew] [vscale] [voffset] 0 0 1`


Here is a very nice interactive tool for visualizing the effect of the various values:
https://codepen.io/GottZ/full/d73f2f844b52b91b7457febce2d1b18c/


Apply the calibration matrix by saying

`xinput set-prop '<device name>' 'Coordinate Transformation Matrix' 1.04 0 -0.02 0 1.04 -0.02 0 0 1`

I just experimented with the values until I had the scree calibrated, it only 
took a few iterations to get so the pointer was exactly under were I touched.


References:
https://wiki.archlinux.org/index.php/Calibrating_Touchscreen
https://wiki.ubuntu.com/X/InputCoordinateTransformation


[1]: https://www.freedesktop.org/wiki/Software/xinput_calibrator/
[2]: https://github.com/notro/fbtft/issues/445
