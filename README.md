racoon-ipsec-osx Cookbook
=========================

Manages racoon, the IKE key management daemon used for IPSec on Mac OS
X (and other platforms).

Specifically, this cookbook manages the racoon configuration and
service to override timeout values that OS X autogenerates. Reference
[this Apple discussion thread](https://discussions.apple.com/thread/3275811?start=0&tstart=0)
for more information/background.

Note that the forum post "accepted answer" includes options that are
not managed here, as it was not required to use those options for
functionality on my system.

This cookbook does not manage IPSec VPN configuration. Use the fancy
OS X gui for that.

Requirements
============

Platform: Mac OS X

Tested on Mac OS X 10.8.3. May work on other versions with or without
modification.

Attributes
==========

* `node['racoon']['proposal_lifetime']` - Lifetime proposed
  for phase 1 negotiations. Default is 186 hours to workaround OS X
  generated configuration. See `racoon.conf(5)` for more information.
  Used in the `racoon_ipsec_osx` resource.

Resources
=========

This cookbook provides the `racoon_ipsec_osx` resource. This is an OS
X specific configuration based on the default generated by OS X for
IPSec VPNs. It is responsible for rendering a configuration file for a
given VPN connection and restarting the racoon service.

**Note**: This is very much tied to the way that OS X generates the
  racoon configurations for IPSec connections.

## Actions

* `:create` - Creates the configuration
* `:delete` - Deletes the configuration

## Attributes

* `ipaddress` - **Name attribute**. The IP address of the remote VPN
  server to configure.
* `source` - Source filename. Default: `racoon-remote.conf.erb`.
* `cookbook` - Cookbook where the source template is. Default: nil
  (uses the `racoon-remote.conf.erb` from this cookbook).
* `my_identifier` - Specifies the identifier sent to the remote host.
  Typically the name of the IPSec shared group.
  Uses the `keyid_use` type. Required.
* `shared_secret` - Specifies the ID from the OS X keychain for the
  shared secret. See below. Required.
* `xauth_login` - Specifies the login to use in client-side hybrid
  authentication. Required.
* `encryption_algorithms` - The encryption algorithms to use for each
  proposal. The default value is a hash that comes from the defaults
  generated by OS X, which should be sufficient for most use cases.

This doesn't manage the actual IPSec configuration. That should be
done through the OS X Network UI. Once complete, the `shared_secret`
will be in the OS X Keychain.

## Example

    racoon_ipsec_osx "192.168.173.12" do
      my_identifier "hqipsec"
      shared_secret "11223344-5566-7788-99AA-BBCCDDEEFF11.SS"
      xauth_login "vpnuser"
    end

Recipes
=======

## default.rb

This recipe ensures that the `/etc/racoon/racoon.conf` config file is
managed with the appropriate content and that the `racoon` service is
managed.

## data-bag-config.rb

This recipe uses a data bag driven configuration, so potentially
sensitive data about the IPSec connection can be separated from the
attributes and recipes in this cookbook. See __Usage__/__Data Bag__
for details.

Usage
=====

Include `recipe[racoon-ipsec-osx]` on a node to manage the basics, and
then use the resource `racoon_ipsec_osx` in a site-specific cookbook
with the appropriate values for any IPSec configuration needed for
Racoon.

To use the data bag driven configuration, include
`recipe[racoon-ipsec-osx::data-bag-config]` recipe on a node, and
create the data bag per the instructions below.

## Data Bag

Create a data bag named `racoon_ipsec` with an item named `default`.
It should have content like this:

    {
      "id": "default",
      "ipaddress": "192.168.173.12",
      "my_identifier": "hqipsec",
      "shared_secret": "11223344-5566-7788-99AA-BBCCDDEEFF11.SS",
      "xauth_login": "vpnuser"
    }

Change the values as appropriate. Get the `shared_secret` from the
IPSec shared secret entry in the OS X keychain.

Contributing
============

1. Fork the repository on Github
2. Create a named feature branch (like `add_component_x`)
3. Write your change
4. Write tests for your change
5. Run the tests, ensuring they all pass
6. Submit a Pull Request using Github

License and Authors
===================

- Author:: Joshua Timberman <joshua@opscode.com>
- Copyright:: Copyright (c) 2013, Opscode, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
