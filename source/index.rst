Mythical blue lake
==============================================

.. figure:: _static/images/blue-lake.png

   `Red Teaming Path on Try Hack Me <https://tryhackme.com/path/outline/redteaming>`_

If an organisation's grove uses Microsoft Windows, you are almost guaranteed to find AD. Microsoft AD is the
dominant suite used to manage Windows domain networks. And because AD is used for Identity and Access Management
of the entire grove, it holds the keys to the kingdom, making it a very likely target for attackers.

.. toctree::
   :maxdepth: 1
   :includehidden:
   :caption: Breaching

   docs/breach/README.md
   OSINT <https://tymyrddin.github.io/red-recon/docs/osint/README.html>
   Phishing <https://tymyrddin.github.io/red-hurdles/docs/phishing/README.html>
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
   :caption: Links

   Red Team <https://tymyrddin.github.io/red/>