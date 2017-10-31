---
id: version-0.33-running-on-device-android
title: running-on-device-android
original_id: running-on-device-android
---
<a id="content"></a><h1><a class="anchor" name="running-on-device"></a>Running On Device <a class="hash-link" href="docs/running-on-device-android.html#running-on-device">#</a></h1><div><h2><a class="anchor" name="prerequisite-usb-debugging"></a>Prerequisite: USB Debugging <a class="hash-link" href="docs/running-on-device-android.html#prerequisite-usb-debugging">#</a></h2><p>You'll need this in order to install your app on your device. First, make sure you have <a href="https://www.google.com/search?q=android+Enable+USB+debugging" target="_blank">USB debugging enabled on your device</a>.</p><p>Check that your device has been <strong>successfully connected</strong> by running <code>adb devices</code>:</p><div class="prism language-javascript">$ adb devices
List of devices attached
emulator<span class="token number">-5554</span> offline   # Google emulator
14ed2fcc device         # Physical device</div><p>Seeing <strong>device</strong> in the right column means the device is connected. Android - go figure :) You must have <strong>only one device connected</strong>.</p><p>Now you can use <code>react-native run-android</code> to install and launch your app on the device. If you get a "bridge configuration isn't available" error, see the <code>Using adb reverse</code> section below.</p><h2><a class="anchor" name="setting-up-an-android-device"></a>Setting up an Android Device <a class="hash-link" href="docs/running-on-device-android.html#setting-up-an-android-device">#</a></h2><p>Let's now set up an Android device to run our React Native projects.</p><p>First thing is to plug in your device and check the manufacturer code by using <code>lsusb</code>, which should output something like this:</p><div class="prism language-javascript">$ lsusb
Bus <span class="token number">002</span> Device <span class="token number">002</span><span class="token punctuation">:</span> ID <span class="token number">8087</span><span class="token punctuation">:</span><span class="token number">0024</span> Intel Corp<span class="token punctuation">.</span> Integrated Rate Matching Hub
Bus <span class="token number">002</span> Device <span class="token number">001</span><span class="token punctuation">:</span> ID 1d6b<span class="token punctuation">:</span><span class="token number">0002</span> Linux Foundation <span class="token number">2.0</span> root hub
Bus <span class="token number">001</span> Device <span class="token number">003</span><span class="token punctuation">:</span> ID 22b8<span class="token punctuation">:</span><span class="token number">2e76</span> Motorola PCS
Bus <span class="token number">001</span> Device <span class="token number">002</span><span class="token punctuation">:</span> ID <span class="token number">8087</span><span class="token punctuation">:</span><span class="token number">0024</span> Intel Corp<span class="token punctuation">.</span> Integrated Rate Matching Hub
Bus <span class="token number">001</span> Device <span class="token number">001</span><span class="token punctuation">:</span> ID 1d6b<span class="token punctuation">:</span><span class="token number">0002</span> Linux Foundation <span class="token number">2.0</span> root hub
Bus <span class="token number">004</span> Device <span class="token number">001</span><span class="token punctuation">:</span> ID 1d6b<span class="token punctuation">:</span><span class="token number">0003</span> Linux Foundation <span class="token number">3.0</span> root hub
Bus <span class="token number">003</span> Device <span class="token number">001</span><span class="token punctuation">:</span> ID 1d6b<span class="token punctuation">:</span><span class="token number">0002</span> Linux Foundation <span class="token number">2.0</span> root hub</div><p>These lines represent the USB devices currently connected to your machine.</p><p>You want the line that represents your phone. If you're in doubt, try unplugging your phone and running the command again:</p><div class="prism language-javascript">$ lsusb
Bus <span class="token number">002</span> Device <span class="token number">002</span><span class="token punctuation">:</span> ID <span class="token number">8087</span><span class="token punctuation">:</span><span class="token number">0024</span> Intel Corp<span class="token punctuation">.</span> Integrated Rate Matching Hub
Bus <span class="token number">002</span> Device <span class="token number">001</span><span class="token punctuation">:</span> ID 1d6b<span class="token punctuation">:</span><span class="token number">0002</span> Linux Foundation <span class="token number">2.0</span> root hub
Bus <span class="token number">001</span> Device <span class="token number">002</span><span class="token punctuation">:</span> ID <span class="token number">8087</span><span class="token punctuation">:</span><span class="token number">0024</span> Intel Corp<span class="token punctuation">.</span> Integrated Rate Matching Hub
Bus <span class="token number">001</span> Device <span class="token number">001</span><span class="token punctuation">:</span> ID 1d6b<span class="token punctuation">:</span><span class="token number">0002</span> Linux Foundation <span class="token number">2.0</span> root hub
Bus <span class="token number">004</span> Device <span class="token number">001</span><span class="token punctuation">:</span> ID 1d6b<span class="token punctuation">:</span><span class="token number">0003</span> Linux Foundation <span class="token number">3.0</span> root hub
Bus <span class="token number">003</span> Device <span class="token number">001</span><span class="token punctuation">:</span> ID 1d6b<span class="token punctuation">:</span><span class="token number">0002</span> Linux Foundation <span class="token number">2.0</span> root hub</div><p>You'll see that after removing the phone, the line which has the phone model ("Motorola PCS" in this case) disappeared from the list. This is the line that we care about.</p><p><code>Bus 001 Device 003: ID 22b8:2e76 Motorola PCS</code></p><p>From the above line, you want to grab the first four digits from the device ID:</p><p><code>22b8:2e76</code></p><p>In this case, it's <code>22b8</code>. That's the identifier for Motorola.</p><p>You'll need to input this into your udev rules in order to get up and running:</p><div class="prism language-javascript">echo SUBSYSTEM<span class="token operator">==</span><span class="token string">"usb"</span><span class="token punctuation">,</span> ATTR<span class="token punctuation">{</span>idVendor<span class="token punctuation">}</span><span class="token operator">==</span><span class="token string">"22b8"</span><span class="token punctuation">,</span> MODE<span class="token operator">=</span><span class="token string">"0666"</span><span class="token punctuation">,</span> GROUP<span class="token operator">=</span><span class="token string">"plugdev"</span> <span class="token operator">|</span> sudo tee <span class="token operator">/</span>etc<span class="token operator">/</span>udev<span class="token operator">/</span>rules<span class="token punctuation">.</span>d<span class="token operator">/</span><span class="token number">51</span><span class="token operator">-</span>android<span class="token operator">-</span>usb<span class="token punctuation">.</span>rules</div><p>Make sure that you replace <code>22b8</code> with the identifier you get in the above command.</p><p>Now check that your device is properly connecting to ADB, the Android Debug Bridge, by using <code>adb devices</code>.</p><div class="prism language-javascript">List of devices attached
TA9300GLMK    device</div><h2><a class="anchor" name="accessing-development-server-from-device"></a>Accessing development server from device <a class="hash-link" href="docs/running-on-device-android.html#accessing-development-server-from-device">#</a></h2><p>You can also iterate quickly on device using the development server. Follow one of the steps described below to make your development server running on your laptop accessible for your device.</p><blockquote><p>Hint</p><p>Most modern android devices don't have a hardware menu button, which we use to trigger the developer menu. In that case you can shake the device to open the dev menu (to reload, debug, etc.). Alternatively, you can run the command <code>adb shell input keyevent 82</code> to open the dev menu (82 being the <a href="http://developer.android.com/reference/android/view/KeyEvent.html#KEYCODE_MENU" target="_blank">Menu</a> key code).</p></blockquote><h3><a class="anchor" name="using-adb-reverse"></a>Using adb reverse <a class="hash-link" href="docs/running-on-device-android.html#using-adb-reverse">#</a></h3><blockquote><p>Note that this option is available on devices running android 5.0+ (API 21).</p></blockquote><p>Have your device connected via USB with debugging enabled (see paragraph above on how to enable USB debugging on your device).</p><ol><li>Run <code>adb reverse tcp:8081 tcp:8081</code></li><li>You can use <code>Reload JS</code> and other development options with no extra configuration</li></ol><h3><a class="anchor" name="running-packager-on-a-non-standard-port"></a>Running packager on a non-standard port <a class="hash-link" href="docs/running-on-device-android.html#running-packager-on-a-non-standard-port">#</a></h3><p>Launch the packager manually with <code>react-native start --port [PORT]</code></p><p>For Android: Use the developer menu by clicking the menu button or shake. Select 'Debug server host &amp; port for device' to set a different port.</p><span><center>
  <img src="img/AndroidDeveloperMenu.png" width="162">
  <img src="img/AndroidDevSettings.png" width="150">
  <img src="img/AndroidDevServerDialog.png" width="150">
</center>

</span><p>For IOS: Edit the AppDelegate.m and change the line below to match the port number you're running:</p><p><code>jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.ios.bundle?platform=ios&amp;dev=true"];</code></p><h3><a class="anchor" name="configure-your-app-to-connect-to-the-local-dev-server-via-wi-fi"></a>Configure your app to connect to the local dev server via Wi-Fi <a class="hash-link" href="docs/running-on-device-android.html#configure-your-app-to-connect-to-the-local-dev-server-via-wi-fi">#</a></h3><ol><li>Make sure your laptop and your phone are on the <strong>same Wi-Fi network</strong>.</li><li>Open your React Native app on your device. You can do this the same way you'd open any other app.</li><li>You'll see a red screen with an error. This is OK. The following steps will fix that.</li><li>Open the <strong>Developer menu</strong> by shaking the device or running <code>adb shell input keyevent 82</code> from the command line.</li><li>Go to <code>Dev Settings</code>.</li><li>Go to <code>Debug server host for device</code>.</li><li>Type in your machine's IP address and the port of the local dev server (e.g. 10.0.1.1:8081). <strong>On Mac</strong>, you can find the IP address in System Preferences / Network. <strong>On Windows</strong>, open the command prompt and type <code>ipconfig</code> to find your machine's IP address (<a href="http://windows.microsoft.com/en-us/windows/using-command-line-tools-networking-information" target="_blank">more info</a>).</li><li>Go back to the <strong>Developer menu</strong> and select <code>Reload JS</code>.</li></ol></div><div class="docs-prevnext"><a class="docs-next" href="docs/signed-apk-android.html#content">Next →</a></div><p class="edit-page-block">You can <a target="_blank" href="https://github.com/facebook/react-native/blob/master/docs/RunningOnDeviceAndroid.md">edit the content above on GitHub</a> and send us a pull request!</p><div class="survey"><div class="survey-image"></div><p>Recently, we have been working hard to make the documentation better based on your feedback. Your responses to this yes/no style survey will help us gauge whether we moved in the right direction with the improvements. Thank you!</p><center><a class="button" href="https://www.facebook.com/survey?oid=516954245168428">Take Survey</a></center></div>