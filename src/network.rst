Network configuration
=====================


Firewall setup (UFW)
--------------------

By default, UFW drops all forwarding traffic. As a result you will need to
enable UFW forwarding:

::
    sudo editor /etc/default/ufw
    # Change:
    # DEFAULT_FORWARD_POLICY="DROP"
    # to
    DEFAULT_FORWARD_POLICY="ACCEPT"
