## Chef Server Populator

Creates orgs, clients, and admin users and installs provided public keys. Simplifies managing and
recreating Chef Server nodes. Provides backup and restore recipes for
complete Chef Server recovery.

### New Chef 12 Support
Chef 12 is supported in version 1.0 and above. If you need Chef 11
support, please pin your environment to version 0.4.0.

### Usage

**When bootstrapping with the chef-server cookbook and chef-solo:**

* Download and unpack chef-server, chef-server-ingredient, packagecloud, and chef-server-populator cookbooks
* Upload public keys to be used by users, org-validator, and clients (optionally)
* Create json for organization, user, and (optionally) client(s)
* Run chef-solo

See the `default[:chef_server_populator][:solo_org]` and
`default[:chef_server_populator][:solo_org_user]` attribute hashes in
`attributes/default.rb` for the required attribute structure.

**When converging with chef-client:**

* Create data bag to hold data bag items with user, org, and client information
* Create data bag items with user, org, and client information
* Set data bag related attributes

Applicable attributes:

* `node[:chef_server_populator][:databag]` - name of the data bag

Structure of the data bag item:

User: 
```json
{
  "id": "user_name",
  "chef_server": {
    "full_name": "User Name",
    "email": "name@domain.tld",
    "client_key": "public key contents",
    "type": [
      "user"
    ],
    "orgs": {
      "organization": {
        "enabled": true,
        "admin": true
      }
   } 
}
```
Note: While users can belong to multiple organizations, and the above
hash structure allows you to define multiple associations, the
chef-server-populator currently only supports the first organization
that is defined in the data bag.

Client:
```json
{
  "id": "client_name",
  "chef_server": {
    "client_key": "public key contents",
    "type": [
      "client"
    ],
    "orgs": [ "organization" ]
   } 
}
```
Note: If no organization is specified for a client, it will be added
to the default organization. The client `enabled` and `admin` settings
can be set at the top level of the `chef_server` hash or in and `orgs`
hash as in the User example.

Org:
```json
{
  "id": "org_name",
  "chef_server": {
    "full_name": "Organization Name",
    "client_key": "public key contents",
    "type": [
      "org"
    ],
    "enabled": true
  }
}
```
Note: Creating the org will create a client called `<org>-validator` which uses the public key specified when 
creating the org.
In addition, there is currently a bug in Chef server 12.1 which means only the first word in
the full name will be used, as the option is not parsed correctly

**Restoring from a backup:**

* Set path to restore file with node[:chef_server_populator][:restore][:file]
* The restore recipe is run if a restore file is set
* The restore file can be remote or local

**When enabling backups:**

* Include chef-server-populator::restore recipe
* Set backup cron interval with node[:chef_server_populator][:schedule]
* Optionally set a remote storage location with node[:chef_server_populator][:backup][:remote][:connection]
* Backups include both a pg_dump of the entire chef database and a tarball of the Chef data directory

## Public Key Format

The format of the public key specified with the json object needs to be a single line string with new lines
represented with the \n character

You can use one of the below commands to convert your public key file into the correct string format (credit
to the certificates cookbook for these)
```
cat <filename> | sed s/$/\\\\n/ | tr -d '\n'
-OR-
/usr/bin/env ruby -e 'p ARGF.read' <filename>
-OR-
perl -pe 's!(\x0d)?\x0a!\\n!g' <filename>
```
If you need to obtain the public key string for your private key first, then run the following on the .pem
file containing the private key
```
openssl rsa -in <path_to_keyfile>.pem -pubout
```

## Extras

Need to use the IP address of the node for a bit, or another name  instead of
having `node[:fqdn]`?

* `node[:chef_server_populator][:servername_override]`

Keep chef server configured via chef client:

* `node[:chef_server_populator][:chef_server]`

If the hash is non-empty, it will write the chef-server `dna.json` and trigger a
`reconfigure` when ever the attributes are updated.

## Known Issues

* As mentioned above, user and client data bag items currently only
  support the first organization provided. Multi-org support is
  forthcoming.

## Examples

Take a look in the `examples` directory for basic bootstrap templates that will
build a new erchef server, using existing keys and client, and
register itself, or restore an existing chef server from a backup.

## Info
* Repository: https://github.com/hw-cookbooks/chef-server-populator
* IRC: Freenode @ #heavywater
