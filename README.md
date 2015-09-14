This wiki describes how to configure Prosody as XMPP back end for ZCS
Prosody Configuration
=====================
Configure authentication
-------------------------
<ol>
<li>Obtain mod_auth_zimbra.lua for your version of Prosody. <br/>Currently, we have two versions of mod_auth_zimbra.lua, one for Prosody 0.8x and one for Prosody 0.9x. </li>
<li>Copy mod_auth_zimbra.lua to Prosody modules folder e.g. <pre> /usr/lib/prosody/modules </pre></li>
<li>Enable "auth_zimbra" authentication mechanism<br/>
You will need to edit prosody configuration file, e.g.: 
<pre>/etc/prosody/prosody.cfg.lua</pre>
Prosody allows configuring authentication globally and per domain, so all of the following options can be added to prosody.cfg.lua either under VirtualHost section or above it. <br/>The following options enable authentication against ZCS SOAP interface. 
<br/>Replace "domain.com" with the name of your ZCS domain
<br/>Replace "admin@domain.com" with username of a ZCS admin account that has read permissions to any user account in the domain
<br/>Replace "test123" with the ZCS admin's password
<pre>
authentication = "zimbra"
zimbra_admin = "admin@domain.com"
zimbra_admin_pw = "test123"
zimbra_host_port = "domain.com"
zimbra_admin_host_port = "domain.com:7071"
zimbra_proto = "https"
zimbra_domain = "domain.com"
</pre>
</li>
</ol>

Configure BOSH
--------------
Zimbra Chat UI uses Bidirectional-streams Over Synchronous HTTP (BOSH) to transport XMPP stanzas (http://xmpp.org/extensions/xep-0206.html). BOSH is usually not enabled by default, so you need to enable BOSH module in Prosody configuration. 
Find "modules_enabled" section in prosody.cfg.lua. It looks like this:
<pre>
-- This is the list of modules Prosody will load on startup.
-- It looks for mod_modulename.lua in the plugins folder, so make sure that exists too.
-- Documentation on modules can be found at: http://prosody.im/doc/modules
modules_enabled = {

        -- Generally required
                "roster"; -- Allow users to have a roster. Recommended ;)
                "saslauth"; -- Authentication for clients and servers. Recommended if you want to log in.
                "tls"; -- Add support for secure TLS on c2s/s2s connections
                "dialback"; -- s2s dialback support
                "disco"; -- Service discovery
                "posix"; -- POSIX functionality, sends server to background, enables syslog, etc.
</pre>
....
<pre>
        -- Other specific functionality
                "isolate_host";
                --"groups"; -- Shared roster support
                --"announce"; -- Send announcement to all online users
                --"welcome"; -- Welcome users who register accounts
                --"watchregistrations"; -- Alert admins of registrations
                --"motd"; -- Send a message to users when they log in
                --"legacyauth"; -- Legacy authentication. Only used by some old clients and bots.
};
</pre>

Find HTTP modules section
<pre>
        -- HTTP modules
                --"bosh"; -- Enable BOSH clients, aka "Jabber over HTTP"
                --"http_files"; -- Serve static files from a directory over HTTP
</pre>
and uncomment "bosh" module
<pre>
        -- HTTP modules
                "bosh"; -- Enable BOSH clients, aka "Jabber over HTTP"
                --"http_files"; -- Serve static files from a directory over HTTP

</pre>
Disable user registration
--------------------------
Add this line below "modules_enabled" section:
<pre>allow_registration = false;</pre>
Allow NGINX to connect to BOSH without SSL
------------------------------------------
When Zimbra Web Client is running with HTTPS and BOSH is running without HTTPS, you will need to add the following option to prosody.cfg.lua:
<pre>consider_bosh_secure = true</pre>

Multiple Domains
==================
Adding domains
------------------
As was already mentioned in [[Configure authentication]] section, Prosody supports multiple domains via Virtual Hosts section in prosody.cfg.lua.
In order to configure authentication for multiple domains, add authentication options under <b>VirtualHost</b> section instead of globally. 

Keep in mind that every time you add a domain, you have to restart Prosody.

Following is an example of configuring 2 domains on the same machine:

<pre>
----------- Virtual hosts -----------
-- You need to add a VirtualHost entry for each domain you wish Prosody to serve.
-- Settings under each VirtualHost entry apply *only* to that host.

VirtualHost "ubuntu2.local"

authentication = "zimbra"
zimbra_admin = "admin@ubuntu2.local"
zimbra_admin_pw = "test123"
zimbra_host_port = "my-zimbra-server"
zimbra_admin_host_port = "my-zimbra-server:7071"
zimbra_proto = "https"
zimbra_domain = "ubuntu2.local"

VirtualHost "ubuntu3.local"
authentication = "zimbra"
zimbra_admin = "admin@ubuntu3.local"
zimbra_admin_pw = "test123"
zimbra_host_port = "my-zimbra-server"
zimbra_admin_host_port = "my-zimbra-server:7071"
zimbra_proto = "https"
zimbra_domain = "ubuntu3.local"
</pre>
Preventing cross-domain communication
-------------------------------------
Currently not possible. A Prosody modules exist that claims to isolate Virtual Hosts, however, cross domain contact requests still go through.
