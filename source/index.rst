Mythical blue lake
==============================================

If an organisation's grove uses Microsoft Windows, you are almost guaranteed to find AD. Microsoft AD is the
dominant suite used to manage Windows domain networks. And because AD is used for Identity and Access Management
of the entire grove, it holds the keys to the kingdom, making it a very likely target for attackers.

----

.. toctree::
   :maxdepth: 1
   :includehidden:
   :caption: Breaching

   docs/breach/README.md
   OSINT <https://red.tymyrddin.dev/projects/recon/en/latest/docs/osint/README.html>
   Phishing <https://red.tymyrddin.dev/projects/fire/en/latest/docs/phishing/README.html>
   docs/breach/ntlm.md
   docs/breach/ldap.md
   docs/breach/relays.md
   docs/breach/mdt.md
   docs/breach/config.md

.. toctree::
   :maxdepth: 1
   :includehidden:
   :caption: Enumerating

   docs/enum/README.md
   docs/enum/setup.md
   docs/enum/injection.md
   docs/enum/mmc.md
   docs/enum/cmd.md
   docs/enum/powershell.md
   docs/enum/bloodhound.md
   docs/enum/cleanup.md

.. toctree::
   :maxdepth: 1
   :includehidden:
   :caption: Lateral movement and pivoting

   docs/pivot/README.md
   docs/pivot/setup.md
   docs/pivot/moving.md
   docs/pivot/spawning.md
   docs/pivot/lateral.md
   docs/pivot/auth.md
   docs/pivot/behaviour.md
   docs/pivot/portforward.md
   docs/pivot/cleanup.md

.. toctree::
   :maxdepth: 1
   :includehidden:
   :caption: Exploiting

   docs/exploit/README.md
   docs/exploit/setup.md
   docs/exploit/permissions.md
   docs/exploit/kerberos.md
   docs/exploit/relays.md
   docs/exploit/users.md
   docs/exploit/gpos.md
   docs/exploit/certificates.md
   docs/exploit/trusts.md

.. toctree::
   :maxdepth: 1
   :includehidden:
   :caption: Persisting

   docs/persist/README.md
   docs/persist/setup.md
   docs/persist/creds.md
   docs/persist/tickets.md
   docs/persist/certs.md
   docs/persist/sid.md
   docs/persist/group.md
   docs/persist/acls.md
   docs/persist/gpos.md

.. toctree::
   :maxdepth: 1
   :includehidden:
   :caption: Credentials harvesting

   docs/harvest/README.md
   docs/harvest/access.md
   docs/harvest/lwc.md
   docs/harvest/lsass.md
   docs/harvest/wcm.md
   docs/harvest/dc.md
   docs/harvest/laps.md
   docs/harvest/hashes.md
