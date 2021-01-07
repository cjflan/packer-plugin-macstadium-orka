Packer Builder Plugin for Orka
==============================

This is a [Packer Builder] plugin to automate building images for [MacStadium Orka],
a Kubernetes/Docker-based macOS virtualization PaaS/SaaS service by [MacStadium].

![example screenshot](./images/screenshot1.jpg)

### Compatibility

For this plugin to function you need to have at least Packer 1.6.0 installed and Orka CLI 1.3.0.

 * [Packer Downloads] - 1.6.0+
 * [Orka CLI Downloads] - 1.3.0+

## Install / Setup

1. Install [Packer](https://www.packer.io/downloads.html)
2. Install [Orka CLI](https://orkadocs.macstadium.com/docs/downloads)
3. Setup Orka CLI - See: [Orka Setup Guide]
3. Download the [Latest Release] of this plugin.
4. Rename the binary to `packer-builder-macstadium-orka`, then move it to a location where [Packer] will detect them at run-time, such as any of the following:
    * The directory where the [packer] binary is.
    * The `~/.packer.d/plugins` directory.
    * The current working directory.
5. Ensure that you make this downloaded file executable and that it runs on your machine.  For OS-X you may need to remove a the quarantine bit with the following...
    ```bash
    # On every Unix-ey OS you'll need to chmod it
    chmod a+x ~/.packer.d/plugins/packer-builder-macstadium-orka
    # On Macs because of a security system in place, you will need to un-quarantine it
    xattr -d ~/.packer.d/plugins/com.apple.quarantine/packer-builder-macstadium-orka
    # If you try to run it on every os it should work but report that you shouldn't run plugins directly
    ~/.packer.d/plugins/com.apple.quarantine/packer-builder-macstadium-orka
    panic: Please do not execute plugins directly. Packer will execute these for you.
    ```
  6. Change to a directory where you have [packer] templates, and packer run as usual, but using the `macstadium-orka` as the builder, per the example below.

## Packer Builder Configuration

```json
{
  "builders": [{
    "type": "macstadium-orka",
    "source_image": "name-of-image-from-vm-images-list",
    "image_name": "destination-image-name"
  }]
}
```
---

* `type` _(string)_ **(required)**

Must be `macstadium-orka`

* `source_image` _(string)_ **(required)**

This is the source image (vm config) we will be using to launch the VM from.  This should be an entry from `orka vm configs`.  If you don't have one or need to create one, here's an example.  The "base-image" in the below command is looked up from `orka image list`.  See [examples below](#Example-Commands)

* `image_name` _(string)_ (optional)

This is the destination name of the image that will be created.  The image will be located inside `orka image list` when completed.  If not specified this will be autogenerated to the following: `packer-{{unix timestamp}}`

### Development / Internal Options

If you're NOT a developer working on this software you can ignore the following.

But, if you are building/editing/updating this software, you may want to turn on most or all of the following options.  See the [examples/macos-catalina.json](./examples/macos-catalina.json).

* `simulate_create` _(boolean)_ (optional) _*- for devs*_

This is for internal development purposes, to prevent having to constantly create VMs for testing/development of this plugin.  Not for normal use.  The simulated "example" is hardcoded into the code on line 90 of [builder/orka/step_orka_create.go](./builder/orka/step_orka_create.go).  Feel free to edit during local development for your environment, but do not submit a commit/MR editing this please.

* `do_not_delete` _(boolean)_ (optional) _*- for devs*_

By default this plugin automatically deletes the VM afterwards if all scripts ran successfully.  AFAIK there is no mechanism built-into Packer to "force" it to not delete the source VM afterwards, so this fills that gap.  This is useful for debugging builds that are being weird, but is generally not intended for noraml use.  You should just use the resulting image.

* `do_not_image` _(boolean)_ (optional) _*- for devs*_

By default this plugin automatically creates an image of the VM after any provisioning
steps. AFAIK there is no mechanism built-into Packer to "force" it to not image the VM
afterwards, so this fills that gap. This is useful for debugging builds that are being
weird, but is generally not intended for noraml use.

## Example Orka Commands

These aren't directly related to this plugin, exactly, but they're a bit of a simplified
guide to get you started. For a more full guide see: [Orka Setup Guide].

```bash
# Create a config which is used for source_image above
orka vm create-config -v macos-catalina-10-15-5 -c 3 --C 3 --vnc --base-image macos-catalina-10.15.5.img -y

# In essence, this plugin automates running the following 3 commands...
# Start a VM (using the config above)
orka vm deploy -v macos-catalina-10-15-5 --vnc -y

# Save a VM's disk to a disk image
orka image save -v <vmid-here-from-orka-vm-list> -b <destination-image-name> -y

# Stop and remove a VM
orka vm delete --vm <vmid-here-from-orka-vm-list> -y

# Later to launch future images with this image create a new config to launch...
orka vm create-config -v my-new-packerified-vm -c 3 --C 3 --vnc --base-image <destination-image-name> -y
# And launch it...
orka vm deploy -v my-new-packerified-vm --vnc -y
# Or, alternatively if you're done working with an image and want to delete it...
orka image delete --image <destination-image-name> -y
```

## Changelog / History

 * [v1.0.0] - Initial Public Release
   * Basic functionality wrapper around Orka CLI
   * Setup Github Action to automatically build and release

## Development / Bugs / Support

If you want to help contribute to this plugin or find a bug, please [file an issue] on Github, and/or submit me a PR.  This is Open Source, so don't expect me to fix bugs immediately, but I'll try my best to reasonably support this plugin.  Contributors are always welcome though.

To get a development environment up will need a recent golang installed and setup.  Then with a single make command below command it will build, install, and try to run the example at [examples/macos-catalina.json](./examples/macos-catalina.json).  You may need to edit this file to have the source VM config that you have locally, as it is hardcoded to my environment's image `macos-catalina-10-15-5` at the moment.

```bash
make fresh
```

Finally, if you want to support me or this project in some way, please donate to a local animal shelter or makerspace or one of many open-source projects that you rely on regularly.

## Todo (in no particular order)

These are a list of things that are pending to accomplish within' this repo.  Contributors welcome, I might do some of these also, eventually.

 * Add tests (should do this first though, as this is important to keep this plugin functional and debugging issues)
 * Add image management features into the builder instead of having to manage out-of-band with orka cli (eg: delete image)
 * Migrate to the (as of yet) undocumented Orka API.  Maybe MacStadium will help and provide me some documentation?  This will simplify the code greatly, and remove the reliance on the Orka CLI to be pre-installed and pre-configured.  Although, arguably this may make this more complicated because then you need to specify the API Key/Token/Account information into this plugin.  And/or make this plugin read from the same location that the Orka CLI reads it configuration from.  A bit to explore and unpack and improve here.
 * Consider implementing for this plugin to automatically create a VM Config (before launching a VM, and possibly after tied to the image just created)
 * Random thought related to the immediate above feature, maybe this plugin to be able to scan configs and/or images available and automatically use the first one it finds, possibly creating a new config specifically for the image?  If it'd make it easier?  I would need to explore and discuss this with some other users if this would be useful and how best to do it.
 * Improve the JSON parsing code.  JSON parsing in Golang is quite messy, and I'm not particularly good at it yet, so the code for parsing it is very messy.  See: line 137-176 of [builder/orka/step_orka_create.go](./builder/orka/step_orka_create.go) and look at the function `ExtractIPHost` in that file as well.  Could be much improved and hopefully simplified.  Contributors welcome!!!
 * Clean up / improve code / catch more sharp edges and edge-cases, deal with any issues filed on Github.
 * One day, get this merged upstream into Packer as an official builder

## Original Author / License

**Please Note:** I am not associated with, affiliated, or tied to MacStadium in any way other than being a fan of their service.  They did not endorse the creation or support of this plugin (yet).  This software was written as a personal open-source contribution to the community.  If you create work based on this work, please attribute that in your code.

This plugin is "very-loosely" based-on and took inspiration from the [Packer Null Builder], [Packer LXD Builder], and the [Packer Builder Veertu Anka].

* Written by [Farley Farley] ( farley _at_ neonsurge **dawt** com ), [Thomas Farvour]
* License Terms: [GNU GPL v3]

[//]: <> (Ignore, below here are links for ease-of-use above)
[Packer]: https://www.packer.io/
[Packer Builder]: https://www.packer.io/docs/extending/custom-builders.html
[MacStadium]: https://www.macstadium.com
[MacStadium Orka]: https://www.macstadium.com/orka
[Orka]: https://www.macstadium.com/orka
[Packer Downloads]: https://www.packer.io/downloads.html
[Orka CLI Downloads]: https://orkadocs.macstadium.com/docs/downloads
[Orka Setup Guide]: https://orkadocs.macstadium.com/docs/quick-start
[Latest Release]: https://github.com/lumoslabs/packer-builder-macstadium-orka/releases
[Farley Farley]: https://github.com/andrewfarley
[Thomas Farvour]: https://github.com/farvour
[GNU GPL v3]: https://choosealicense.com/licenses/gpl-3.0/
[v1.0.0]: https://github.com/andrewfarley/packer-builder-macstadium-orka/releases/tag/v1.0.0
[SSH Communicator]: https://www.packer.io/docs/communicators/ssh
[Packer Builder Veertu Anka]: https://github.com/veertuinc/packer-builder-veertu-anka
[Packer Null Builder]: https://github.com/hashicorp/packer/tree/master/builder/null
[Packer LXD Builder]: https://github.com/hashicorp/packer/tree/master/builder/lxd
[file an issue]: https://github.com/lumoslabs/packer-builder-macstadium-orka/issues
