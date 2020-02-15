# Automating Kerberos Authentication

Sometimes you need unattended authentication. Sometimes you are just lazy. Whatever the reason, if a user \(human or otherwise\) wants to fetch a Ticket Granting Ticket \(TGT\) from a Kerberos Key Distribution Center \(KDC\) automatically, the Global Security Services API \(GSSAPI\) library shipped with most recent distributions support it. Kerberos is based on symmetric cryptography. If a user needs to store a symmetric key in a filesystem, she uses a file format known as a Key table, or keytab for short. Fetching a keytab is not a standard action, but FreeIPA has shipped with a utility to make it easier: ipa-getkeytab Before I attempt to get a keytab, I want to authenticate to my KDC and get a TGT manually:

```text
1    $ kinit ayoung@YOUNGLOGIC.NET
2    Password for ayoung@YOUNGLOGIC.NET: 
3    [ayoung@ayoung530 tempest (master)]$ klist
4    Ticket cache: KEYRING:persistent:14370:krb_ccache_H4Ss9cA
5    Default principal: ayoung@YOUNGLOGIC.NET
6     
7    Valid starting       Expires              Service principal
8    05/01/2015 09:07:06  05/02/2015 09:06:55  krbtgt/YOUNGLOGIC.NET@YOUNGLOGIC.NET
```

To fetch a keytab and store it in the users home directory, you can run the following command. I’ve coded it to talk to my younglogic.net KDC, so modify it for yours.

```text
1    ipa-getkeytab -p $USER@YOUNGLOGIC.NET -k $HOME/client.keytab -s ipa.younglogic.net
```

You can get your own principal from the klist output:

```text
1    export KRB_PRINCIPAL=$(klist | awk ‘/Default principal:/ {print $3}’)
If you are running on an ipa-client enrolled machine, much of the info you need is in /etc/ipa/default.conf.
01    $ cat   /etc/ipa/default.conf 
02    #File modified by ipa-client-install
03     
04    [global]
05    basedn = dc=younglogic,dc=net
06    realm = YOUNGLOGIC.NET
07    domain = younglogic.net
08    server = ipa.younglogic.net
09    host = rdo.younglogic.net
10    xmlrpc_uri =  [https://ipa.younglogic.net/ipa/xml](https://ipa.younglogic.net/ipa/xml) 
11    enable_ra = True
```

You can convert these values into environment variables with:

```text
1    $(awk ‘/=/ {print “export IPA_” toupper($1)”=“$3}’ < /etc/ipa/default.conf)
```

Now a user could manually kinit using that keytab and the following commands:

```text
1    $(awk ‘/=/ {print “export IPA_” toupper($1)”=“$3}’ < /etc/ipa/default.conf)
2    kinit -k -t $HOME/client.keytab $USER@$IPA_REALM
```

We can skip the kinit step by putting the keytab in a specific location. If you look inthe man page for krb5.conf you can find the following section: /default\_client\_keytaname/ /This relation specifies the name of the default keytab for obtaining client credentials. The default is FILE:/var/kerberos/krb5/user/%euid/client.keytab. This relation is subject to parameter expansion / What is %euid? It is the numeric userid for a user. For yourself, the value is set in $EUID. What if you need it for a different user? Use the getent command to configure the name service switch configured database for this value:

```text
1    $ getent passwd ayoung
2    ayoung:*:622800001:622800001:Adam Young:/home/ayoung:/bin/sh
It is that third value. Again, if you want to automate:
1    export AYOUNG_EUID=getent passwd ayoung | cut -d: -f3
You need to create that directory before you can put something in it. You only want the user to be able to read or write in that directory.
1    sudo mkdir  /var/kerberos/krb5/user/$EUID
2    sudo chown $USER:$USER  /var/kerberos/krb/$EUID 
3    chmod 700  /var/kerberos/k5/user/$EUID
Now use that to store the keytab:
1    ipa-getkeytab -p $KRB_PRINCIPAL -k   /var/kerberos/krb5/user/$EUID/client.keytab -s $IPA_SERVER
To test out the new keytab, kdestroy to remove the existing TGTS then try performing an action that would require a service ticket. 
Here I show an initially cleared credential cache that gets automatically populated when I connect to a remote system via ssh.
01    [ayoung@ayoung530 tempest (master)]$ kdestroy -A
02    [ayoung@ayoung530 tempest (master)]$ klist -A
03    [ayoung@ayoung530 tempest (master)]$ ssh -K rdo.younglogic.net
04    Last login: Fri May  1 16:42:28 2015 from c-1-2-3-4.imadethisup.net
05    -sh-4.2$ exit
06    logout
07    Connection to rdo.younglogic.net closed.
08    [ayoung@ayoung530 tempest (master)]$ klist -A
09    Ticket cache: KEYRING:persistent:14370:krb_ccache_WotXvlm
10    Default principal: ayoung@YOUNGLOGIC.NET
11     
12    Valid starting       Expires              Service principal
13    05/01/2015 12:42:46  05/02/2015 12:42:45  host/rdo.younglogic.net@YOUNGLOGIC.NET
14    05/01/2015 12:42:46  05/02/2015 12:42:45  host/rdo.younglogic.net@
15    05/01/2015 12:42:45  05/02/2015 12:42:45  krbtgt/YOUNGLOGIC.NET@YOUNGLOGIC.NET
```

I would not recommend doing this for normal users. But for service users that need automated access to remote services, this is the correct approach.

