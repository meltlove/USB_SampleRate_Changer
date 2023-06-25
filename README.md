## USB (including a 3.5mm jack and an internal speaker) sample rate changer for Android devices on the fly

This is not a Magisk module but a collection of root shell scripts using Magisk su or phh's SuperUser (uninstall "termux" and magisk modules replacing "toybox" and "busybox" commands if you've used them because they break the Adnroid standard and make this script work wrongly), and can be installed by just unpacking its ZIP archive on your Internal Storage ("/sdcard") or another. Under Magisk environment (<strong>"root name space mount mode" must be changed to "global"</strong> in the settings of Magisk Manager), this script changes the sample rate of the USB (including hardware offloading) audio class driver on Android devices on the fly like the developer setting of Bluetooth LDAC or the Windows mixer for <q><em>avoiding annoying SRC (Sample Rate Conversion) distortion ultimately (i.e., not only the distortion of the OS mixer as usual, but also the over-sampling distortion in DAC's)</em></q>. This script signals the audioserver on the "global" mount name space to try to reload an audio policy configuration file generated by this script with a specified sample rate and bit depth, so the "root name space mount mode" change is needed. <br/>
<br/>
Additionally, this script disables DRC (Dynamic Range Control, i.e., a kind of compression) if DRC has been enabled on a stock firmware. For example, smart phones and tablets whose SoC's have an SDM??? or SM???? model number usually enable DRC on all audio outputs, but those whose SoC's have an MT???? model number don't enable DRC on any audio output. (Note: some custom ROM's (not stocks and GSI's) intendedly invert or change the DRC mode on context, then this script cannot disable DRC)<br/>
<br/>
Finally, the Android OS mixer (AudioFlinger) always apply resampling even to 1:1 ratio pass-through (e.g., 44.1kHz to 44.1kHz resampling with Kaiser windowed digital low-pass filtering; a resampler is consiting of an interpolator to the analogue space (limited analogue values needed only for the following sampler are computed) with an anti-aliasing low-pass filter and a digital sampler by a specified output frequency from the anlogue space to the digital one. The output frequency can be the same as the input one), so you need to be careful for resampling parameters even when resampling is not needed (see the description of "extras/change-resampling-quality.sh" below).

* Usage: `sh /sdcard/USB_SampleRate_Changer/USB_SampleRate_Changer.sh [--reset] [--drc] [--bypass-offload][--bypass-offload-safer][--offload][--offload-hifi-playback][--offload-drect][--legacy][--safe][-safest][--usb-only] [[44k|48k|88k|96k|176k|192k|353k|384k|706k|768k] [[16|24|32|float]]]`,

if you unpack the archive under "/sdcard" (Internal Storage). The arguments are a sample rate and a bit depth (or 32bit float) to which you want to change, respectively. Their default values are `44k` (sample rate: 44.1 kHz) and `32` (bit depth: 32 bits) except both safe and safest mode internal outputs (default values: 48 kHz, 32 or 16 bits).

  - Options
    - `--reset`(without arguments): resets its previous execution results.
    - `--drc`: enables DRC (Dynamic Range Control, i.e., compression, typically sticking to all audio outputs on Qcomm devices) for the purpose of comparison to this script's usual DRC-less audio quality (not effective for --usb-only mode). No DRC unless this option. Please unplug your headphones or earphones before executing a command for propagating this effect.
    - `--bypass-offload`: changes an audio policy configuration file for bypassing USB & Bluetooth hardware offloading (worse in audio quality) and using a non- hardware offloading USB & Bluetooth audio driver (better in audio quality; A small jitter USB driver and an AOSP Bluetooth driver without unnecessary resampling) while a 3.5mm jack and an internal speaker use a hardware offloading driver.
    - `--bypass-offload-safer`: changes an audio policy configuration file for bypassing USB & Bluetooth hardware offloading (worse in audio quality) and using a non- hardware offloading USB & Bluetooth audio driver (better in audio quality) while a 3.5mm jack and an internal speaker use a hardware offloading driver keeping the same settings (48kHz & 16bit/24bit).
    - `--offload`: changes an audio policy configuration file for USB & Bluetooth hardware offloading (worse in audio quality; a large jitter USB driver and a Bluetooth driver forcing unnecessary resampling, i.e., typically 44.1kHz -> 48kHz -> 44.1kHz double resampling). 
    - `--offload-hifi-playback`: changes an audio policy configuration file for USB hardware offloading including a USB audio "hifi_playback" mixer (worse in audio quality; fixed at 192kHz/96kHz 32bit mode typically) while disabling a2dp hardware offloading (better in audio quality). 
    - `--offload-direct`: changes an audio policy configuration file for USB & Bluetooth hardware offloading (worse in audio quality) with "direct_pcm" and "compressed_offload" modes for 3.5mm jack, bluetooth and USB audio output bypassing mixers. Added for comparing usual mixer modes to direct modes on Qcomm devices with&without DRC, especially to bluetooth audio "direct_pcm" mode with&without DRC.
    - `--legacy`: changes an audio policy configuration file for a Bluetooth audio legacy HAL (<em>/system/{lib,lib64}/hw/audio.a2dp.default.so</em>).
    - `--safe`: changes an audio policy configuration file for a Bluetooth audio legacy HAL but keeps considerably traditional settings for an internal speaker and others.
    - `--safest`: changes an audio policy configuration file for a Bluetooth audio legacy HAL but keeps most traditional settings for an internal speaker and others.
    - `--usb-only`: changes a USB audio policy configuration file only.
    - `--force-bluetooth-qti`: forces to use "bluetooth_qti" bluetooth module (Qcomm's legacy one) instead of usual ones.

    - For typical example, `sh /sdcard/USB_SampleRate_Changer/USB_SampleRate_Changer.sh` automatically investigates your device and determines the audio policy configuration type ("offload" (including USB & Bluetooth), "bypass-offload-safer" ("offload" except USB & Bluetooth), "legacy" ("offload" except USB & Bluetooth but using a legacy Bluetooth module "a2dp"), "safe" (for non-offloading devices using the "a2dp") and "safest" (for old devices). And this sets the sample rate and the bit depth of your device to be 44.1 kHz and 32 bits except internal ouputs of "bypass-offload-safer", "safe" and "safest" modes. If you want to set another sample rate and bit depth, please specify specific values.

    - I recommend using `sManager` (Script Manager) or the like for easiness (at boot automatic execution, saving many combinations of script options and parameters as aliases, and so on.

    - When using a USB DAC, please disconnect the DAC from the USB socket of an Android device once after your execute this script every time, for ensuring the changed audio configuration to be effective completely on the device.

* This script and following ones in "extras" folder have been tested on LineageOS and ArrowOS ROM's, and phh GSI's (Android 10 ~ 13, Qualcomm & MediaTek SoC, and Arm32 & Arm64 combinations).

* Note 1: "USB_SampleRate_Changer.sh" requires unlocking the USB audio class driver's limitation (upto 96kHz (USB HAL) lock, 384kHz offload lock for Qcomm devices or 96kHz offload lock for MTK and Tensor devices (actually only bypass to the USB HAL driver)) if you want to specify greater than 96kHz or 384kHz (in case of Qcomm USB hardware offloading, i.e., maybe hardware offload tunneling to the ALSA driver). See my companion magisk module ["usb-samplerate-unlocker"](https://github.com/Magisk-Modules-Alt-Repo/usb-samplerate-unlocker) for non- hardware offload drivers. Although you specify a high sample rate for this script execution, you cannot connect your device to a USB DAC with the sample rate unless the USB DAC supports the sample rate (the USB driver will limit the connecting sample rate down to its maximum sample rate).

* Note 2: This script and other extras ones can be executed under "phh's SuperUser" root environment on recent A/B partition phh-GSI's. If you find some errors under the environment, try `setenforce 0` with root permission for making Selinux mode to be permissive. Some phh-GSI's have wrong selinux settings.

* Note 3: Entry class USB DAC's usually adopt an interface chip communicating with the adaptive mode or the synchronous one defined in the USB audio standard. As in these modes an Android host controller sends audio sampling rate clock signals to the DAC, jitter generated at the host side affects the audio quality of the DAC tremendously. Higher class DAC's communicate with the asynchronous mode (also defined in the standard) to a host controller, but they actually use a PLL to reduce jitter from the host not to stutter even in heavy jitter situations. As this result, they behave as the adaptive mode with a feedback loop to dynamically adjust the host side sampling clock signals while referring a DAC side clock in a real sense, so even with asynchronous mode they are more or less affected by host side jitter. You can see the mode of your USB DAC by opening "/proc/asound/card1/stream0" on your phone while playing music. Please see a word in parentheses at "Endpoint:" lines; "SYNC", "ADAPTIVE" or "ASYNC" means that your DAC uses "synchronous", "adaptive" or "asynchronous" mode to communicate to your phone, respectively. Moreover, almost all audio peripherals, e.g., bluetooth earphones, internal DAC's, network audio devices have a PLL in themselves and are affected by host side jitter for the same reason.

* Tips 1: You can see the sample rate connecting to a USB DAC during music replaying by a command `cat /proc/asound/card1/pcm0p/sub0/hw_params` (for non- USB hardware offload drivers). You can also see mixer ("AudioFlinger") info by a command `dumpsys media.audio_flinger`. There are corresponding convenient scripts ("alsa-hw-params.sh" and "dumpsys-filtered.sh") and others in "extras" folder.

  - Usage:  `sh /sdcard/USB_SampleRate_Changer/extras/alsa-hw-params.sh`
    - outputs information of the ALSA audio driver for USB DAC's, 3.5mm Jack and internal speakers.

  - Usage:  `sh /sdcard/USB_SampleRate_Changer/extras/dumpsys-filtered.sh [--all][--help]`
     - outputs the active peripheral's information from `dumpsys media.audio_flinger`. With `--all` option, this script outputs all peripheral's information from the command.
     
  - Usage:  `sh /sdcard/USB_SampleRate_Changer/extras/dumpsys-bluetooth-filtered.sh [--all][--help]`
     - outputs the active bluetooth codec information from `dumpsys bluetooth_manager`. With `--all` option, this script outputs all bluetooth manager information from the command.

  - Usage:  `sh /sdcard/USB_SampleRate_Changer/extras/change-bluetooth-hal.sh [--help][--status][aosp | legacy | offload | sysbta]"`
     - changes the active bluetooth audio HAL to one you specify. With `--status` option, this script outputs currently enabled bluetooth audio HALs.

  - Usage:  `sh /sdcard/USB_SampleRate_Changer/extras/getConfig.sh [--all][--help]`
    - outputs brief information of the active audio policy configuration. With `--all` option, this script outputs all the information of the configuration.

  - Usage:  `sh /sdcard/USB_SampleRate_Changer/extras/change-usb-period.sh [--status] [--reset][period_usec]`
    - changes the USB audio data transfer period to a specified value in usec (default: 2500usec). With "--status" option, this script outputs the current period without changing the period. With "--reset" option, this script reverts the period to the original (5000usec). The best value is usuall around 2500usec (or 3250usec), but 20375usec (or 2500usec) for POCO F3 and 3875usec for low performance old devices. Since the value directly affects the jitter of a PLL in a USB DAC, this script may improve the audio quality of the DAC frighteningly.

  - Usage:  `sh /sdcard/USB_SampleRate_Changer/extras/change_resampling_quality.sh [--help] [--status] [--reset] [--bypass] [--cheat] [stop_band_dB [half_filter_length [cut_off_percent]]]`
    - changes the resampling quality of the OS mixer (AudioFlinger). "--help" and "--status" options specify printing above usage and the status of AudioFlinger's resampling configuration without configuration changes, respectively. With "--reset" option, this script clears previous settings. "--bypass" option specifies to apply this configuration change except toward less than 48kHz (excluding 48kHz itself) frequencies. "--cheat" option specifies to switch the mode of third argument "cut_off_percent" (if specified) from "cut off" (i.e., at -3dB point, and also the middle point of a transition band) to "transition band width cheat" (i.e., the start point of a stop band). "stop_band_dB", "half_filter_length" and "cut off percent" specify a stop band attenuation in dB, the number of input data needed before the current point (optional) and a cut off (or a transition band width cheat) percent of the Nyquist frequency (optional), respectively. AOSP standard values are 90dB and 32 (cut off: 100%), but this script's default values are 160dB and 480 (cut off: 91%) as mastering quality values, i.e., no resampling distortion in a real sense (even though the 160dB targeted attenuation is not accomplished in the AOSP implementation).
    - Note: the resampler of the Android OS mixer is a pure Kaiser windowed Sinc interpolator (a 16 bit integer polyphase FIR filter and a 32bit one) that has 127 fixed phase sets of FIR coefficients generated from specified parameters and a variable phase set of FIR coeffcients linearly interpolated from other phase sets. So 2<sup>&#177;n</sup> (phases) type up-sampling and down-sampling are very precise.
    - Remark: the Android OS mixer always apply resampling even to 1:1 ratio pass-through (actually digital low-pass filtering), e.g., 44.1kHz to 44.1kHz pass-though (i.e., low-pass filtering). To be bit-perfect pass-through, you need to consider parameters carefully for making a Kaiser window to be one pulse under 16bit, 24bit or 32bit precision. With "--bypass" option, 1:1 ratio resampling from 44.1kHz & 16bit data to 44.1kHz 16bit one keeps bit-perfect (but not to 44.1kHz 24bit or 32bit one). I recommend using 194dB 520 (cut off: 100%) for 1:1 ratio resampling. The half filter length 520 can make an effective jitter buffer and reduce jitter to a certain extent. However this resampling makes audibly large aliasing distortion for none 1:1 ratio resampling under Android 11 (except latest) or earlier because of an aliasing processing bug. For this reason, I also don't recommend specifying "cut off" and "cheat" around and over the Nyquist frequency under Android 11 or earlier. But actually on latest bug-free Android devices, the best general purpose setting (for variable sampling rate input/output) may be 179dB 408 (cheat: 99%).
    - Appendix (Resampling Parameter Examples) :
    
    
    | Stop band attenuation (dB) | Half filter length | Cut-off (%) | Stop band (%) | Memo |
    | ---: | ---: | ---: | ---: | ---- |
    | AOSP parameters: | - | - | - | - |
    | 90 | 32 | 100 | | Default Quality|
    | 80 | 16 | 100 | | LOW Quality |
    | 84 | 16 | 100 | | MED Quality |
    | 98 | 32 | 100 | | HIGH Quality |
    | Recommended parameters: | - | - | - | - |
    | 160 | 480 | 91 | | This scrips's default, esp. for A11 and earlier |
    | 167 | 368 | | 106 | Low performance devices of A12 and later |
    | 179 | 408 | | 99 | General purpose for A12 and later |
    | 194 | 520 | 100 | | 1:1 resampling for bit-perfect |
    | External examples: | - | - | - | - |
    | 100 | 29 | | 109 | AK4493 (Sharp Roll-Off for N-fold over-sampling) |
    | 120 | 35 | | 110 | ESS 9038PRO (Sharp Roll-Off for N-fold over-sampling) |
    | 98 | 130 | 98.5 | | MacOS Leopard (guess) |
    | 160 | 240 | | 100 | iZotope, No-Alias (guess) |
    | 98 | 64 | | 100 | SoX HQ linear phase (guess) |
    | 170 | 520 | | 100 | SoX VHQ linear phase (guess) |

* Tips 2: "jitter-reducer.sh" in "extras" folder is an interactive tool derived from ["Hifi Maximizer"](https://github.com/yzyhk904/hifi-maximizer-mod) which could reduce jitter distortions in all digital audio outputs relating to SELinux mode, thermal controls, doze (battery saving while idling), CPU&GPU governors, logd servers (to interface to logcat or the like), camera server, I/O scheduling, virtual memory, wifi suspension, battery management and the audio effects framework (to interface to equalizers, virtualizers, visualizers, echo cancelers, automatic gain controls, etc.). In my opinion, jitter distortions reduction is the very key to ultimate hifi audio quality.

  - Usage:  `sh /sdcard/USB_SampleRate_Changer/extras/jitter-reducer.sh [--selinux|++selinux][--thermal|++thermal][--doze|++doze][---governor|++governor][--logd|++logd][--camera|++camera][--io [scheduler [light | m-light | medium | boost]] | ++io][--vm|++vm][--wifi|++wifi][--all|++all][--battery|++battery][--effect|++effect][--status][--help]`

    - each "--" prefixed option except "--status" and "--help" options is an enabler for its corresponding jitter reducer, conversely each "++" prefixed option is a disabler for its corresponding jitter reducer. "--all" option is an alias of all "--" prefixed options except "--effect", "--battery", "--status" and "--help" option, and also "++all" option is an alias of all "++" prefixed options except "++effect" and "++battery" option.
    - "scheduler" specifies an I/O scheduler for I/O block devices (typically "deadline", "cfq" or "noop", but you may specify "*" for automatic best selection) and has optional four modes "light" (for warmer tone), "m-light" (for slightly warmer tone), "medium" (default) and "boost" (for clearer tone).
    - please remember that "--wifi" option is persistent even after reboot, but other options are not.
    - and "--doze" option disables doze (light and deep) of your device at all and its battery charge remaining may fall rapidly while idling.

  - For most "hifi" example,  `sh /sdcard/USB_SampleRate_Changer/extras/jitter-reducer.sh --all --battery --effect --status` enables all jitter reducers including effects framework one and outputs the jitter related statuses. For Bluetooth earphones, you may need to add `--io "*" boost` or `--io "*" m-light` option.  For DLNA transmitting, you may need to add `--io "*" boost` option. (If you use "AirMusic" to transmit audio data, I recommend setting around 4597 msec additional delay to reduce jitter distortion on the AirMusic panel to display target device(s).)

* Tips 3 (Convenient): Please use another magisk module of mine ["Audio jitter silencer"](https://github.com/Magisk-Modules-Alt-Repo/audio-jitter-silencer) with this module. I don't recommend this, but for your convenience. This module works automatically (but not completely) as following my recommendation.

* Tips 3 (Recommended): Please disable "Manage apps automatically" in "Battery manager" (or "Adaptive battery" of "Adaptive preferences") in the battery section (needless to say, don't enable battery savers, performance limiters and the like), turn off "Adaptive connectivity" in the Network & internet section (if exists), and change "Battery optimization" from "Optimize" to "Don't optimize" (or change "Battery usage" from "Optimized" to "Unrestricted") for following app's manually through the settings UI of Android OS (to lower less than 10Hz jitter making extremely short reverb or foggy sound like distortion) even though disabling the Android doze itself. music (streaming) player apps, their licensing apps (if exist), "AirMusic" (if exists), "AirMusic  Recording Service" (system app; if exists), equalizer apps (if exist), "Bluetooth" (system app), "Bluetooth MIDI Service" (system app), "MTP Host" (system app), "NFC Service" (system app; if exists), "sManager" or the like (if exists), "Magisk" (if exists), System WebView apps (system app), Browser apps, "PhhTrebleApp" (system app; if exists), "Android Services Library" (system app), "Android Shared Library" (system app), "Android System" (system app), "System UI" (system app), "Input Devices" (system app), {Gesture, 3 Button, 2  Button} Navigation Bar apps (which you are using only; system app), "crDroid System" (system app; if exists), "LineageOS System" (system app; if exists), launcher app, "Google Play Store" (system app), "Google Play services" (system app), "Styles & wallpaper" or the like (system app), {Lineage, crDroid, Arrow, etc.} themes app (system app; if exists),  "AOSP panel" (system app; if exists), "OmniJaws" (system app; if exists), "OmniStyle" (system app; if exists), "Active Edge Service" (system app; if exists), "Android Device Security Module" (system app; if exists), "Call Management" (system app; if exists), "Phone" (system app; if exists), "Phone Calls" (system app; if exists), "Phone Services" (system app; if exists), "Phone and Messaging Storage" (system app; if exists), "Storage Manager" (system app), "Default" (system app; if exists), "Default StatusBar" (system app; if exists), "Wfd Service" (system app; if exists), "Wallpaper" or the like (system app), "Adreno Graphics Drivers" (system app; if exists), "com.android.providers.media" (system app), "Files by Google" (system app; if exists), "Google Play Services for AR" (system app; if exists), "Google Services Framework" (system app), "Waterfall cutout" (system app), "Network Manager" (system app), "Companion Device Manager" (system app), "Intent Filter Verification Service" (system app), "Calendar", camera apps, keyboard app, kernel adiutors (if exist), etc. And uninstall "Digital Wellbeing" (system app; if it exists) itself or change "Battery usage" from "Optimized" to "Restricted" (this is very harmful for audio like camera servers).

* Tips 4: See also my magisk module ["audio-misc-settings"](https://github.com/Magisk-Modules-Alt-Repo/audio-misc-settings). This can increase the number of media volume steps to 100 steps and so on.
<br/>

## DISCLAIMER

* I am not responsible for any damage that may occur to your device, so it is your own choice to attempt this script.

##
