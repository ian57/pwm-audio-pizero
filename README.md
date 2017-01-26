# No audio output on the Pi zero! Really

Yes! to keep the Pi Zero small and low cost, the headphone audio filter isn't included. No jack connection to get some music from your recalbox. You can still get digital audio out via HDMI so if you plug it your Pi into a monitor with speakers, that will work fine.

It's ok if you plugged your recalbox on your TV in your living-room, but in an arcade cabinet with a standard PC monitor screen, or a TV... you can't get sound from the HDMI.

This is not really true. If you use an HDMI-VGA converter, you could be lucky to have one which have a jack audio output as this one https://www.amazon.com/MOKiN-Adapter-Converter-Powered-Laptop/dp/B01F1T9IS4/ref=pd_lpo_147_tr_t_3/153-2977990-6201028?ie=UTF8&psc=1&refRID=GV8D910Z9HMW8BW4FR0P

But if you use a Pi zero, you certainly want to integrate it in a small device, with a small TFT screen on SPI/I2C/DPI bus, and you do not have space for this converter.

# How audio is made on Raspberry Pi

As discribed in the page https://learn.adafruit.com/introducing-the-raspberry-pi-zero/audio-outputs, the others Raspberry Pi use 2 PWM outputs with a filtering circuit to create the sound output. On this page you will find the real electonic device embedded in the Raspberry Pi to create sound.

But, if you read carefully it uses the PWM on the GPIO #40 and #45 which are not brought out on the Pi Zero. 

The solution consists in getting to PWM0 on GPIO #18 (ALT5) and PWM1 on GPIO #13 (ALT0) or GPIO #19 (ALT5) and create your own filtering circuit.

The manipulation is discribed in details in the page https://learn.adafruit.com/adding-basic-audio-ouput-to-raspberry-pi-zero/pi-zero-pwm-audio

With recalbox, the simpler way is to add this line in the `config.txt` file. It will reconfigure the pins at boot without any external software or services. The PWMO will be on the GPIO #18 (pin 12 on the connector), and PWM1 on GPIO #13 (pin 33 on the connector).

```ini
dtoverlay=pwm-2chan,pin=18,func=2,pin2=13,func2=4
```

As described in the page https://hackaday.io/project/9467-piboy-zero/log/35090-pi-zero-pwm-audio-device-tree-overlay, you can make you own overlay with the following source code


```c
/dts-v1/;
/plugin/;

/ {
  compatible = "brcm,bcm2708";

  fragment@0 {
    target = <&gpio>;
    __overlay__ {
      pinctrl-names = "default";
      pinctrl-0 = <&pwm_audio_pins>;

    pwm_audio_pins: pwm_audio_pins {
	brcm,pins = <13 18>;   /* gpio no ('BCM' number) */
	brcm,function = <4 2>; /* 0:in, 1:out, 2: alt5, 3: alt4, 4: alt0, 5: alt1, 6: alt2, 7: alt3 */
	brcm,pull = <0 0>;     /* 2:up 1:down 0:none */
      };
    };
  };
 };
```

If you have setup buildroot as described in https://github.com/recalbox/recalbox-os/wiki/Compilation-%26-Modifications-%28EN%29 and already built recalbox, you can compile the dts with :

```bash
output/build/linux-FIRMWARE/scripts/dtc/dtc -@ -I dts -O dtb -o pwm-audio-pi-zero-overlay.dtbo pwm-audio-pi-zero-overlay.dts
```
and you copy the pwm-audio-pi-zero-overlay.dtbo file in /boot/overlays of your recalbox. The configuration is now simpler : 
```ini
dtoverlay=pwm-audio-pi-zero
```
#Quick test of the sound output

You can test if it works without creating the filtering circuit. You can connect a headphone speaker directly between the PWM pin and a ground as chown in the next figure. 

It should work but the sound can have flaw. 

#Enhance the sound output

In order to create your filter you'll need some electronic components : 		
-* 2 x 270 Ohms resistances		
-* 2 x 150 Ohms ressitances		
-* 2 x 10µF capacitors		
-* 2 x 33nF capacitors (can be replaced by 22nF or 10nF capacitors)		
		
Here is the schematics		
		
![audioFilter_bb.png](http://images.morere.eu/audioFilter_bb.png)		
		
and the real filter		
		
![audioFilter.jpg](http://images.morere.eu/audioFilter.jpg)		
		
You'll need to amplify the output because the signals are filtered and have very low levels.		
		
# Amplification and volume control		
		
You can use a small 5V 2x3W amplificator as this one http://www.ebay.fr/itm/121952326706?trksid=p2060353.m2749.l2649&ssPageName=STRK%3AMEBIDX%3AIT. You can add a an stereo audio potentiometer to be able to tune the volume.		
		
![audioAmpli.jpg](http://images.morere.eu/audioAmpli.jpg)


Same page https://github.com/recalbox/recalbox-os/wiki/Analog-Audio-Pi-Zero-%28EN%29 for details
