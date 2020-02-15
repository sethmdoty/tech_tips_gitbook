# Centrify AD join Issues

1\)adleave -f —user svctask —password 1qaz@WSX

2\)\#rm /etc/krb5/krb5.conf or /etc/krb5.conf

3\) \#rm /etc/krb5/krb5.keytab or /etc/krb5.keytab

4\) \#rm etc/krb5/krb5.ccache or /etc/krb5.ccache

5\) \#adinfo —diag \( make sure all ports for the domain controllers are open \)

SHN=`/bin/hostname | cut -d’.’ -f1` /usr/sbin/adjoin —force —user svctask —password 1qaz@WSX -z HAPI\_PROD -n ${SHN} hammocks.local NOTE: prewin2k name should be set with -N force machine key update: adkeytab -r -u $Domain\_Admin

