---
title: "Sed: Extract Patterns From Files and Texts"
date: 2023-01-16T17:21:27+03:00
authors: ["Gokberk Gunes", ]
tags: ["scripting",]
draft: false
---

In this article, extracting a pattern from a text file and from another output
will be demonstrated. In order to do this we will use `sed` though this is not
the only approach to this problem. The importance of this problem is that users
face with similar text manipulation very frequently especially when they write
shell scripts.

## Getting a Synthetic Data
Consider you run the following command, and you would like to get the card's name.

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

### Turning Selected Line to Regex
This piece of text is actually the output of Wireplumber's inspect
command on my machine with some truncated parts. Now, suppose that we want to
extract what `alsa.card_name` is equal to. First, we need to  generate the
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

This is the regex equivalent of the line we selected.
```txt
\s*alsa\.card_name = "\(.*\)"
```
### Extracting the Regex Line
Only thing left is to call `sed` command. We are passing `-n` to suppress
printing of the non-matching lines. `s/` at start will substitute our matching
pattern with `\1` which refers to first group. `/p` at the end will print the
matching line for us.

## Getting the Desired Pattern
The most common ways to extract patters are from a file and from another
commands output. Both are the extractions with `sed` are very straightforward
to apply.
### Extract Pattern From a File
Here we directly call `sed` to find desired regex pattern from a file called
**file.txt**.
```sh
sed -n 's/\s*alsa\.card_name = "\(.*\)"/\1/p' file.txt
```

### Extract Pattern From Pipe
Below, we are extracting the output of some other command called `program`. In
order to do this, we pipe the output to `sed` with `|` symbol.
```sh
program | sed -n 's/\s*alsa\.card_name = "\(.*\)"/\1/p'
```


