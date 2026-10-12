# sidetone-toggle

A small script that toggles **mic sidetone** on Linux, so you can hear your own voice in your headphones during calls. It uses PipeWire's (or PulseAudio's) `module-loopback` and KDE's `kdialog` for menus and notifications.

Built for a USB ModMic with headphones on a separate DAC, but it works with any input and output.

## Requirements

- PipeWire with `pipewire-pulse` (or PulseAudio)
- `pactl`
- `kdialog`

On Arch Linux:

```sh
sudo pacman -S pipewire-pulse libpulse kdialog
```

## Install

```sh
mkdir -p ~/.local/bin
curl -o ~/.local/bin/sidetone-toggle \
  https://raw.githubusercontent.com/loganmarchione/sidetone-toggle/main/sidetone-toggle
chmod +x ~/.local/bin/sidetone-toggle
```

Make sure `~/.local/bin` is on your `PATH`.

### Keyboard shortcut (KDE)

To bind the script to a key, go to **System Settings → Keyboard → Shortcuts → Add New → Command or Script**, enter `sidetone-toggle`, and choose a key.

## Usage

```sh
sidetone-toggle          # toggle sidetone on/off
sidetone-toggle --setup  # pick devices and latency again
```

- **First run:** choose your microphone, headphone output, and latency. Your choices are saved.
- **Later runs:** toggles on and off straight away, with no menus.
- If a saved device isn't connected, the menus appear again.

## How it works

- Sidetone is a `module-loopback` from your mic to your headphones, labeled `media.name=Sidetone`.
- To decide whether to turn sidetone on or off, the script looks for that label among the running modules. No state file is needed, so a PipeWire restart can't confuse it.
- Settings are stored in `~/.config/sidetone-toggle.conf`:

  ```sh
  SOURCE=alsa_input.usb-...
  SINK=alsa_output.usb-...
  LATENCY=1
  ```

  You can edit this file by hand. Delete it to start over.

## Tips

- **Crackling or dropouts:** raise the latency with `--setup` (try 5–10 ms). Run `pw-top` while sidetone is on: `QUANT` shows the real buffer size and `ERR` counts dropouts.
- **Volume:** the loopback appears as **Sidetone** in the Plasma volume applet's Applications tab, so you can turn it down there.
- **Muting:** if you mute inside Discord or Zoom, you'll still hear yourself, because sidetone reads the mic directly.
- **Call apps:** turn off automatic mic volume and keep noise suppression low, so what you hear matches what others hear.
- **Dialog size:** change `GEOMETRY` at the top of the script. If that has no effect on Wayland, set the size with a KWin window rule that matches the window title `Sidetone`.

## Troubleshooting

If sidetone won't turn off, check that the label is present:

```sh
pactl list short modules | grep loopback
```

You should see `media.name=Sidetone` in the arguments. To remove the loopback by hand:

```sh
pactl unload-module <module-id>
```

## Credits

Inspired by Ben Tasker's post [Injecting Audio Sidetone using Pipewire or PulseAudio](https://www.bentasker.co.uk/posts/documentation/linux/adding-mic-monitoring-to-headphone-output-on-linux.html).
