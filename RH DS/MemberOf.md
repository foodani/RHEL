# MemberOf Plugin

Create the custom ObjectClass and custom Attribute.

The attribute to use with the MemberOf plugin must have "DN" syntax.

Enable the plugin

```
dsconf -D "cn=Directory Manager" ldap://server.example.com plugin memberof enable
dsctl instance_name restart
```

To retrieve members of a group from a different attribute than member, which is the default, set the memberOfGroupAttr parameter to the respective attribute name.

For example, to read group members from uniqueMember attributes, replace the current value of memberOfGroupAttr:

Optionally, display the attribute that is currently configured:

```
dsconf -D "cn=Directory Manager" ldap://server.example.com plugin memberof show
...
memberofgroupattr: 
...
```
The command displays that currently only the member attribute is configured to retrieve members of a group.

Add the uniqueMember attribute to the configuration:

```
dsconf -D "cn=Directory Manager" ldap://server.example.com plugin memberof set --groupattr uniqueMember
successfully added memberOfGroupAttr value "uniqueMember"
```

To set multiple attributes, pass them all to the --groupattr parameter. For example:

```
dsconf -D "cn=Directory Manager" ldap://server.example.com plugin memberof set --groupattr member uniqueMember ...
```

By default, the MemberOf plug-in adds the memberOf attribute to user entries. To use a different attribute, set the name of the attribute in the memberOfAttr parameter.

For example, to add the customMemberOf attribute to user records, replace the current value of memberOfAttr:

Optionally, display the attribute that is currently configured:

```
dsconf -D "cn=Directory Manager" ldap://server.example.com plugin memberof show
...
memberofattr: 
...

Configure the MemberOf plug-in to add the customMemberOf attribute to user entries:

```
dsconf -D "cn=Directory Manager" ldap://server.example.com plugin memberof set --attr customMemberOf
memberOfAttr set to "customMemberOf"
```

You can only set this parameter to an attribute that supports DN syntax.

In an environment that uses distributed databases, you can configure the plug-in to search user entries in all databases instead of only the local database:
```
dsconf -D "cn=Directory Manager" ldap://server.example.com plugin memberof set --allbackends on
memberOfAllBackends enabled successfully
```

Restart the instance:

```
dsctl instance_name restart
```
