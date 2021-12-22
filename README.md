# VagrantRoyaleExample

## Summary

Simple example project for using Vagrant to start a server that can host a compiled Apache Royale project.  This was created to help test [Vagrant support](https://github.com/Moonshine-IDE/Moonshine-IDE/issues/770) for Moonshine-IDE.

## Requirements

Here are the requirements for this demo:
- [Moonshine-IDE](https://moonshine-ide.com/) (Optional, but referenced in instructions below)
- [Apache Royale](https://royale.apache.org/download/) (Can be installed with Moonshine-IDE)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Vagrant](https://www.vagrantup.com/downloads)

I only tested this on macOS, so the requirements may be slightly different for Windows.

## Usage

To try the demo on macOS:
1. Open the project in Moonshine by double-clicking VagrantRoyaleExample.as3proj (or `Project > Open/Import Project`)
2. Right-click on the project and choose `Settings`.  Check that the settings are correct for your environment.  Set a custom SDK if you do not use Royale as the default SDK for Moonshine-IDE.
3. Build the project with `Project > Build`
4. Right-click on the project and choose `Copy Path`
5. Open a Terminal window and run `cd %paste-path-from-above%`
6. Start Vagrant with `vagrant up`
7. Wait for the command to complete.  This may take several minutes
8. Open the server in a local browser:  [http://127.0.0.1:8080/index.html].  The path will be provided in the `vagrant up` output
9. If you want to restart the server use `CTRL-C` to kill the Vagrant command and run `vagrant up` again (or `vagrant reload` for a VM restart).  `vagrant up` should finish in a few seconds since the setup is already done.
10. When, you are done with the demo, run `vagrant halt`, and `vagrant destroy` to destroy the virtualbox VM
