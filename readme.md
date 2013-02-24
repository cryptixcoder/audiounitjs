# What is it?

audiounit.js is an Xcode project scaffold for creating audio software (effects, analyzers and synthesizers) for OS X and iOS.  The audio processing code is written in C++, and the UI code is written in HTML, CSS and Javascript.  Writing the audio processing and UI code will produce an iOS app, a Mac app, and a Mac Audio Unit plug-in that will work in professional audio software like Digital Performer, Logic, or Live.

# Why?

Two reasons:

 - For plug-in developers:  writing plug-in UIs is often the most time consuming part of the plug-in creation process.  audiounit.js makes this step easy by using standardized web technologies for the UI.
 - For audio programmers interested in iOS: Writing audio software for iOS is hard due to poor documentation of Audio Units on iOS.  With audiounit.js, your own custom Audio Units can be injected into the iOS audio system.

# What isn't it?

Unfortunately, audiounit.js does not make the actual signal processing code any easier.  You still have to write an Audio Unit in C++, which is not an easy task for a beginner programmer.

# How do I get it?

Because Xcode templates are difficult to write and seem to change with every version, audiounit.js is distributed as a node.js script.  To install, first install node from the [official website](http://nodejs.org).  Then, install audiounit.js with

```bash
sudo npm install -g audiounitjs
```

if you're still confused, there's a [quick screencast](http://youtu.be/tqxOLf8EmdU) on the install process.
        
# How do I use it?

First, create a configuration json file as described below.  Then, run `audiounitjs myfile.json`, which will create a project scaffold for you.  Then it's a simple matter of programming - edit the `audio.cpp` file to write the audio source, and edit the `ui` folder to create your beautiful UI.

# What exactly does it give me?

The Xcode project generated by the script has 3 targets:

 * An Audio Unit plug-in for use in professional audio programs like Digital Performer.  This will work in any Audio Unit host that supports Cocoa UIs, and is fully standards compliant.  All communication between the UI and the Audio Unit happens through the API and there is no shared state, so in theory the audio-processing could occur in a separate process.
 * A native Mac App.  The output and input of your processing will be the user's current default output and input.  All MIDI sources will be routed to your audio code.
 * A native iOS app.  The input will be the device's microphone, and the output can be configured to be the device's speaker or receiver.  All MIDI sources hooked up to the device will be routed to your audio code.

# Examples

Creating a project will provide a minimal example of a volume-changing playthrough effect.  This simply connects the input to the output, adjusted by a user-settable gain.

Two other examples are included in the `examples` directory, each exemplifying an important feature.

<dl>
<dt>fivescope</dt>
<dd>is a vintage-style oscilloscope, showing how to use the properties API (see below) to transfer complex data (like an oscilloscope trace) from the audio processor to the UI.</dd>
<dt>monosine</dt>
<dd>is a simple monophonic sine wave synthesizer.  This shows how to deal with MIDI input and how to send MIDI from the HTML UI.</dd>
</dl>

# Configuration file format:

You can look at the "config_example.json" file to get you started.

<dl>
<dt>project</dt>
<dd>This is the name of the output Xcode project file.  It must be unique as it is used as a class prefix for Cocoa (if it isn't unique, your plug-in may have namespace collisions when other plug-ins are loaded).</dd>
<dt>type</dt>
<dd>The audio unit plug-in type.  Common ones are 'aumu' for synthesizers, 'aumf' for effects that receive MIDI, and 'aufx' for effects that don't receive MIDI.  A full list of types is in the apple document titled "Audio Unit Component Services Reference".</dd>
<dt>plugin_id</dt>
<dd>The ID for your plug-in.  This must be four letters long and unique for each plug-in.</dd>
<dt>manufacturer_id</dt>
<dd>Your manufacturer id.</dd>
<dt>name</dt>
<dd>The name of your plug-in or app.  May contain spaces.</dd>
<dt>description</dt>
<dd>A human-readable description that will be shown in certain plug-in hosts.</dd>
<dt>company</dt>
<dd>Human readable company name that will be shown in plug-in hosts</dd>
<dt>width, height</dt>
<dd>The height and width of your UI for the plug-in and mac app.  The iOS app of course will be the size of the device's screen</dd>
<dt>preferred_iPhone_output</dt>
<dd>Set this to either "Speaker" or "Receiver" depending on where you want the output to happen.</dd>
</dl>

# Javascript API

To write the UI for your program, use the standard web languages: HTML, CSS, and Javascript.  When the program loads, it will automatically navigate to `index.html` in your UI folder.

To communicate with the C++ audio processing code, use the `AudioUnit` javascript object in the global scope.  For each Audio Unit property and parameter, this object contains a subobject.  For example, if you created a property called `Volume` in your C++ code, There will be an object called `AudioUnit.Volume` in your javascript scope.  Parameters refer to settings that can be changed from the javascript code (or plug-in hosts like Digital Performer), while Properties are data that can only be read from the javascript.  Both parameters and properties have the following methods:

 * `Get()` - returns the current value of the property or parameter

and the following fields:

 * `OnChange` - set this to be a function that will be called whenever the parameter or property changes.

Parameters have the following additional methods:

 * `Set(value)` - sets the current value.
 * `BeginGesture()` - notifies the plug-in host that the user has started to change the parameter
 * `EndGesture()` - notifies the plug-in host that the user has ended the current gesture

and the following additional fields:

 * `OnBeginGesture` - set this to a function that will be called whenever the plug-in host starts a gesture.
 * `OnEndGesture` - set this to a function that will be called whenever the plug-in host ends a gesture.
 
so, if you have a parameter called `Volume`, `var v = AudioUnit.Volume.Get();` would return it's current value, while `AudioUnit.Volume.Set(.2);` would set the value to `.2`.

Additionally, the AudioUnit object has three methods of its own for sending MIDI to your Audio Unit:

 * `AudioUnit.NoteOn(note, velocity)` - call this to trigger a note-on event
 * `AudioUnit.NoteOff(note, velocity)` - call this to trigger a note-off event
 * `AudioUnit.SendMIDI(b1, b2, b3)` - call this to send a three-byte MIDI message.

CoreAudio doesn't provide a way for plug-in UIs to listen to MIDI directly, so there's no way to receive MIDI from the javascript code.

# C++ API

To write your C++ audio processing code, simply create an Audio Unit as described by the apple document entitled "Audio Unit Programming Guide".  You can look at the provided examples for some more concrete hints.

In addition to the standard Audio Unit API, audiounit.js provides a simple method for allowing complex properties to be available to the Javascript code.  Simply fill-in the code for `Audio::GetPropertyDescriptionList` with a vector of all properties you want to be available to Javascript.  See the `fivescope` example for more information.

In the non-plugin targets for iOS and Mac, all MIDI received by the system will be sent to your Audio Unit.  MIDI is handled as in the Audio Unit standard - see the `monosine` example for more information.