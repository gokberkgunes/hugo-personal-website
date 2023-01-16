---
title: "Sed: Extract Patterns From Files and Texts"
date: 2023-01-16T17:21:27+03:00
draft: false
---

In this article, extracting a pattern from a text using `sed` and `regex` will
be shown. This process is very straightforward and users face this very
frequently when they write shell scripts.

Consider you runthe following command, and you would like to get the card's name.

```shtop
wpctl inspect @DEFAULT_SINK@
```
{{< highlight text "hl_lines=3" >}}
>id 53, type PipeWire:Interface:Node
>    alsa.card = "1"
>    alsa.card_name = "SteelSeries Arctis 1 Wireless"
>    alsa.class = "generic"
>    alsa.device = "0"
>    alsa.driver_name = "snd_usb_audio"
>    ...
>  * object.path = "alsa:pcm:1:iec958:1:playback"
>  * object.serial = "11163"
>  * priority.driver = "1008"
>  * priority.session = "1008"
{{< /highlight >}}

This piece of text is actually output from Wireplumber's inspect command. Now,
we want to extract what `alsa.card_name` is equal to. First let's generate the
regex equivalent that exact line.

```text
    alsa\.card_name = "SteelSeries Arctis 1 Wireless"
```
The line starts with four spaces, we can match to this with `\s*`, then we have
to match our variable name `alsa\.card_name = "`. The text in the double quotes
is what we aim to get from out command. We can group this piece of information
as `\(.*\)`, here backslashes and braces group whatever inside them dot star
will match any length of any character. Lastly, we will add closing quote to
the pattern.

This is what we have now,
```txt
\s*alsa\.card_name = "\(.*\)"
```
Only thing left is to call sed command. We are passing `-n` to suppress
printing of the non-matching lines. `s/` at start will substitute our matching
pattern with `\1` which refers to first group. `/p` at the end will print the
matching line for us.

Below application of the command for a piped output, 
```sh
other_command | sed -n 's/\s*alsa\.card_name = "\(.*\)"/\1/p'
```
Here is the one for applying the command for a file,
```sh
sed -n 's/\s*alsa\.card_name = "\(.*\)"/\1/p' file.txt
```

