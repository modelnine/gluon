Site configuration
==================

The ``site`` consists of the files ``site.conf`` and ``site.mk``.
In the first community based values are defined, which both are processed
during the build process and runtime.
The last is directly included in the make process of Gluon.

Configuration
-------------

The ``site.conf`` is a lua dictionary with the following defined keys.

hostname_prefix
    A string which shall prefix the default hostname of a device.

site_name
    The name of your community.

site_code
    The code of your community. It is good practice to use the TLD of
    your community here.

prefix4 \: optional
    The IPv4 Subnet of your community mesh network in CIDR notation, e.g.
    ::

       prefix4 = '10.111.111.0/18'

    Required if ``next_node.ip4`` is set.

prefix6
    The IPv6 subnet of your community mesh network, e.g.
    ::

       prefix6 = 'fdca::ffee:babe:1::/64'

timezone
    The timezone of your community live in, e.g.
    ::

      -- Europe/Berlin
      timezone = 'CET-1CEST,M3.5.0,M10.5.0/3'

ntp_server
    List of NTP servers available in your community or used by your community, e.g.:
    ::

       ntp_servers = {'1.ntp.services.ffac','2.ntp.services.ffac'}

    This NTP servers must be reachable via IPv6 from the nodes. If you don't want to set an IPv6 address
    explicitly, but use a hostname (which is recommended), see also the :ref:`FAQ <faq-dns>`.

opkg \: optional
    ``opkg`` package manager configuration.

    There are two optional fields in the ``opkg`` section:

    - ``lede`` overrides the default LEDE repository URL. The default URL would
      correspond to ``http://downloads.lede-project.org/snapshots/packages/%A``
      and usually doesn't need to be changed when nodes are expected to have IPv6
      internet connectivity.
    - ``extra`` specifies a table of additional repositories (with arbitrary keys)

    ::

      opkg = {
        lede = 'http://opkg.services.ffac/lede/snapshots/packages/%A',
        extra = {
          gluon = 'http://opkg.services.ffac/modules/gluon-%GS-%GR/%S',
        },
      }

    There are various patterns which can be used in the URLs:

    - ``%n`` is replaced by the LEDE version codename
    - ``%v`` is replaced by the LEDE version number (e.g. "17.01")
    - ``%S`` is replaced by the target board (e.g. "ar71xx/generic")
    - ``%A`` is replaced by the target architecture (e.g. "mips_24kc")
    - ``%GS`` is replaced by the Gluon site code (as specified in ``site.conf``)
    - ``%GV`` is replaced by the Gluon version
    - ``%GR`` is replaced by the Gluon release (as specified in ``site.mk``)

regdom \: optional
    The wireless regulatory domain responsible for your area, e.g.:
    ::

      regdom = 'DE'

    Setting ``regdom`` is mandatory if ``wifi24`` or ``wifi5`` is defined.

wifi24 \: optional
    WLAN configuration for 2.4 GHz devices.
    ``channel`` must be set to a valid wireless channel for your radio.

    There are currently three interface types available. You many choose to
    configure any subset of them:

    - ``ap`` creates a master interface where clients may connect
    - ``mesh`` creates an 802.11s mesh interface with forwarding disabled
    - ``ibss`` creates an ad-hoc interface

    Each interface may be disabled by setting ``disabled`` to ``true``.
    This will only affect new installations.
    Upgrades will not change the disabled state.

    Additionally it is possible to configure the ``supported_rates`` and ``basic_rate``
    of each radio. Both are optional, by default hostapd/driver dictate the rates.
    If ``supported_rates`` is set, ``basic_rate`` is required, because ``basic_rate``
    has to be a subset of ``supported_rates``.
    The example below disables 802.11b rates.

    ``ap`` requires a single parameter, a string, named ``ssid`` which sets the
    interface's ESSID. This is the WiFi the clients connect to.

    ``mesh`` requires a single parameter, a string, named ``id`` which sets the
    mesh id, also visible as an open WiFi in some network managers. Usually you 
    don't want users to connect to this mesh-SSID, so use a cryptic id that no
    one will accidentally mistake for the client WiFi.

    ``ibss`` requires two parametersr: ``ssid`` (a string) and ``bssid`` (a MAC).
    An optional parameter ``vlan`` (integer) is supported.

    Both ``mesh`` and ``ibss`` accept an optional ``mcast_rate`` (kbit/s) parameter for
    setting the multicast bitrate. Increasing the default value of 1000 to something
    like 12000 is recommended.
    ::

       wifi24 = {
         channel = 11,
         supported_rates = {6000, 9000, 12000, 18000, 24000, 36000, 48000, 54000},
         basic_rate = {6000, 9000, 18000, 36000, 54000},
         ap = {
           ssid = 'alpha-centauri.freifunk.net',
         },
         mesh = {
           id = 'ueH3uXjdp',
           mcast_rate = 12000,
         },
         ibss = {
           ssid = 'ff:ff:ff:ee:ba:be',
           bssid = 'ff:ff:ff:ee:ba:be',
           mcast_rate = 12000,
         },
       },

wifi5 \: optional
    Same as `wifi24` but for the 5Ghz radio.

next_node \: package
    Configuration of the local node feature of Gluon
    ::

      next_node = {
        ip4 = '10.23.42.1',
        ip6 = 'fdca:ffee:babe:1::1',
        mac = 'ca:ff:ee:ba:be:00'
      }

    The IPv4 next-node address is optional.

mesh \: optional
    Options specific to routing protocols.

    At the moment, only the ``batman_adv`` routing protocol has such options:

    The optional value ``gw_sel_class`` sets the gateway selection class. The default
    class 20 is based on the link quality (TQ) only, class 1 is calculated from
    both the TQ and the announced bandwidth.
    ::

       mesh = {
         batman_adv = {
           gw_sel_class = 1,
         },
       }


mesh_vpn
    Remote server setup for the mesh VPN.

    The `enabled` option can be set to true to enable the VPN by default. `mtu`
    defines the MTU of the VPN interface.

    The `fastd` section configures settings specific to the *fastd* VPN
    implementation.

    If `configurable` is set to `false` or unset, the method list will be replaced on updates
    with the list from the site configuration. Setting `configurable` to `true` will allow the user to
    add the method ``null`` to the beginning of the method list or remove ``null`` from it,
    and make this change survive updates. Setting `configurable` is necessary for the
    package `gluon-web-mesh-vpn-fastd`, which adds a UI for this configuration.

    In any case, the ``null`` method should always be the first method in the list
    if it is supported at all. You should only set `configurable` to `true` if the
    configured peers support both the ``null`` method and methods with encryption.

    You can set syslog_level from verbose (default) to warn to reduce syslog output.

    The `tunneldigger` section is used to define the *tunneldigger* broker list.

    **Note:** It doesn't make sense to include both `fastd` and `tunneldigger`
    sections in the same configuration file, as only one of the packages *gluon-mesh-vpn-fastd*
    and *gluon-mesh-vpn-tunneldigger* should be installed with the current
    implementation.

    ::

      mesh_vpn = {
        -- enabled = true,
        mtu = 1280,

        fastd = {
          methods = {'salsa2012+umac'},
          -- configurable = true,
          -- syslog_level = 'warn',
          groups = {
            backbone = {
              -- Limit number of connected peers from this group
              limit = 1,
              peers = {
                peer1 = {
                  key = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX',
                  -- Having multiple domains prevents SPOF in freifunk.net
                  remotes = {
                    'ipv4 "vpn1.alpha-centauri.freifunk.net" port 10000',
                    'ipv4 "vpn1.alpha-centauri-freifunk.de" port 10000',
                  },
                },
                peer2 = {
                  key = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX',
                  -- You can also omit the ipv4 to allow both connection via ipv4 and ipv6
                  remotes = {'"vpn2.alpha-centauri.freifunk.net" port 10000'},
                },
                peer3 = {
                  key = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX',
                  -- In addition to domains you can also add ip addresses, which provides
                  -- resilience in case of dns outages
                  remotes = {
                    '"vpn3.alpha-centauri.freifunk.net" port 10000',
                    '[2001:db8::3:1]:10000',
                    '192.0.2.3:10000',
                  },
                },
              },
              -- Optional: nested peer groups
              -- groups = {
              --   lowend_backbone = {
              --     limit = 1,
              --     peers = ...
              --   },
              -- },
            },
            -- Optional: additional peer groups, possibly with other limits
            -- peertopeer = {
            --   limit = 10,
            --   peers = { ... },
            -- },
          },
        },

        tunneldigger = {
          brokers = {'vpn1.alpha-centauri.freifunk.net'}
        },

        bandwidth_limit = {
          -- The bandwidth limit can be enabled by default here.
          enabled = false,

          -- Default upload limit (kbit/s).
          egress = 200,

          -- Default download limit (kbit/s).
          ingress = 3000,
        },
      }

mesh_on_wan \: optional
    Enables the mesh on the WAN port (``true`` or ``false``).
    ::

       mesh_on_wan = true,

mesh_on_lan \: optional
    Enables the mesh on the LAN port (``true`` or ``false``).
    ::
    
        mesh_on_lan = true,

poe_passthrough \: optional
    Enable PoE passthrough by default on hardware with such a feature.

autoupdater \: package
    Configuration for the autoupdater feature of Gluon.

    The mirrors are checked in random order until the manifest could be downloaded
    successfully or all mirrors have been tried.
    ::

      autoupdater = {
        branch = 'stable',
        branches = {
          stable = {
            name = 'stable',
            mirrors = {
              'http://[fdca:ffee:babe:1::fec1]/firmware/stable/sysupgrade/',
              'http://autoupdate.alpha-centauri.freifunk.net/firmware/stable/sysupgrade/',
            },
            -- Number of good signatures required
            good_signatures = 2,
            pubkeys = {
              'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX', -- someguy
              'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX', -- someother
            }
          }
        }
      }

    All configured mirrors must be reachable from the nodes via IPv6. If you don't want to set an IPv6 address
    explicitly, but use a hostname (which is recommended), see also the :ref:`FAQ <faq-dns>`.

roles \: optional
    Optional role definitions. Nodes will announce their role inside the mesh.
    This will allow in the backend to distinguish between normal, backbone and
    service nodes or even gateways (if they advertise that role). It is up to
    the community which roles to define. See the section below as an example.
    ``default`` takes the default role which is set initially. This value should be
    part of ``list``. If you want node owners to change the role via config mode add
    the package ``gluon-web-node-role`` to ``site.mk``.

    The strings to display in the web interface are configured per language in the
    ``i18n/en.po``, ``i18n/de.po``, etc. files of the site repository using message IDs like
    ``gluon-web-node-role:role:node`` and ``gluon-web-node-role:role:backbone``.
    ::

      roles = {
        default = 'node',
        list = {
          'node',
          'test',
          'backbone',
          'service',
        },
      },

setup_mode \: package
    Allows skipping setup mode (config mode) at first boot when attribute
    ``skip`` is set to ``true``. This is optional and may be left out.
    ::

      setup_mode = {
        skip = true,
      },

legacy \: package
    Configuration for the legacy upgrade path.
    This is only required in communities upgrading from Lübeck's LFF-0.3.x.
    ::

      legacy = {
             version_files = {'/etc/.freifunk_version_keep', '/etc/.eff_version_keep'},
             old_files = {'/etc/config/config_mode', '/etc/config/ffac', '/etc/config/freifunk'},
             config_mode_configs = {'config_mode', 'ffac', 'freifunk'},
             fastd_configs = {'ffac_mesh_vpn', 'mesh_vpn'},
             mesh_ifname = 'freifunk',
             tc_configs = {'ffki', 'freifunk'},
             wifi_names = {'wifi_freifunk', 'wifi_freifunk5', 'wifi_mesh', 'wifi_mesh5'},
      }

Packages
--------

The ``site.mk`` is a Makefile which should define constants
involved in the build process of Gluon.

GLUON_SITE_PACKAGES
    Defines a list of packages which should be installed additionally
    to the ``gluon-core`` package.

GLUON_RELEASE
    The current release version Gluon should use.

GLUON_PRIORITY
    The default priority for the generated manifests (see the autoupdater documentation
    for more information).

GLUON_REGION
    Region code to build into images where necessary. Valid values are the empty string,
    ``us`` and ``eu``.

GLUON_LANGS
    List of languages (as two-letter-codes) to be included in the web interface. Should always contain
    ``en``.

.. _site-config-mode-texts:

Config mode texts
-----------------

The community-defined texts in the config mode are configured in PO files in the ``i18n`` subdirectory
of the site configuration. The message IDs currently defined are:

gluon-config-mode:welcome
    Welcome text on the top of the config wizard page.

gluon-config-mode:pubkey
    Information about the public VPN key on the reboot page.

gluon-config-mode:novpn
    Information shown on the reboot page, if the mesh VPN was not selected.

gluon-config-mode:altitude-label
    Label for the ``altitude`` field

gluon-config-mode:altitude-help
    Description for the usage of the ``altitude`` field

gluon-config-mode:reboot
    General information shown on the reboot page.

There is a POT file in the site example directory which can be used to create templates
for the language files. The command ``msginit -l en -i ../../docs/site-example/i18n/gluon-site.pot``
can be used from the ``i18n`` directory to create an initial PO file called ``en.po`` if the ``gettext``
utilities are installed.

.. note::

   An empty ``msgstr``, as is the default after running ``msginit``, leads to
   the ``msgid`` being printed as-is. It does *not* hide the whole text, as
   might be expected.

   Depending on the context, you might be able to use comments like
   ``<!-- empty -->`` as translations to effectively hide the text.

Site modules
------------

The file ``modules`` in the site repository is completely optional and can be used
to supply additional package feeds from which packages are built. The git repositories
specified here are retrieved in addition to the default feeds when ``make update``
it called.

This file's format is very similar to the toplevel ``modules`` file of the Gluon
tree, with the important different that the list of feeds must be assigned to
the variable ``GLUON_SITE_FEEDS``. Multiple feed names must be separated by spaces,
for example::

    GLUON_SITE_FEEDS='foo bar'

The feed names may only contain alphanumerical characters, underscores and slashes.
For each of the feeds, the following variables are used to specify how to update
the feed:

PACKAGES_${feed}_REPO
    The URL of the git repository to clone (usually ``git://`` or ``http(s)://``)

PACKAGES_${feed}_COMMIT
    The commit ID of the repository to use

PACKAGES_${feed}_BRANCH
    Optional: The branch of the repository the given commit ID can be found in.
    Defaults to the default branch of the repository (usually ``master``)

These variables are always all uppercase, so for an entry ``foo`` in GLUON_SITE_FEEDS,
the corresponding configuration variables would be ``PACKAGES_FOO_REPO``,
``PACKAGES_FOO_COMMIT`` and ``PACKAGES_FOO_BRANCH``. Slashes in feed names are
replaced by underscores to get valid shell variable identifiers.


Examples
--------

site.mk
^^^^^^^

.. literalinclude:: ../site-example/site.mk
  :language: makefile

site.conf
^^^^^^^^^

.. literalinclude:: ../site-example/site.conf
  :language: lua

i18n/en.po
^^^^^^^^^^

.. literalinclude:: ../site-example/i18n/en.po
  :language: po

i18n/de.po
^^^^^^^^^^

.. literalinclude:: ../site-example/i18n/de.po
  :language: po

modules
^^^^^^^

.. literalinclude:: ../site-example/modules
  :language: makefile

site-repos in the wild
^^^^^^^^^^^^^^^^^^^^^^

This is a non-exhaustive list of site-repos from various communities:

* `site-ffa <https://github.com/tecff/site-ffa>`_ (Altdorf, Landshut & Umgebung)
* `site-ffac <https://github.com/ffac/site>`_ (Regio Aachen)
* `site-ffbs <https://github.com/ffbs/site-ffbs>`_ (Braunschweig)
* `site-ffhb <https://github.com/FreifunkBremen/gluon-site-ffhb>`_ (Bremen)
* `site-ffda <https://github.com/freifunk-darmstadt/site-ffda>`_ (Darmstadt)
* `site-ffeh <https://github.com/freifunk-ehingen/site-ffeh>`_ (Ehingen)
* `site-fffl <https://github.com/freifunk-flensburg/site-fffl>`_ (Flensburg)
* `site-ffgoe <https://github.com/freifunk-goettingen/site-ffgoe>`_ (Göttingen)
* `site-ffgt-rhw <https://github.com/ffgtso/site-ffgt-rhw>`_ (Guetersloh)
* `site-ffhh <https://github.com/freifunkhamburg/site-ffhh>`_ (Hamburg)
* `site-ffho <https://git.ffho.net/freifunkhochstift/ffho-site>`_ (Hochstift)
* `site-ffhgw <https://github.com/lorenzo-greifswald/site-ffhgw>`_ (Greifswald)
* `site-ffka <https://github.com/ffka/site-ffka>`_ (Karlsruhe)
* `site-ffki <http://git.freifunk.in-kiel.de/ffki-site/>`_ (Kiel)
* `site-fflz <https://github.com/freifunk-lausitz/site-fflz>`_ (Lausitz)
* `site-ffl <https://github.com/freifunk-leipzig/freifunk-gluon-leipzig>`_ (Leipzig)
* `site-ffhl <https://github.com/freifunk-luebeck/site-ffhl>`_ (Lübeck)
* `site-fflg <https://github.com/kartenkarsten/site-fflg>`_ (Lüneburg)
* `site-ffmd <https://github.com/FreifunkMD/site-ffmd>`_ (Magdeburg)
* `site-ffmwu <https://github.com/freifunk-mwu/sites-ffmwu>`_ (Mainz, Wiesbaden & Umgebung)
* `site-ffmyk <https://github.com/FreifunkMYK/site-ffmyk>`_ (Mayen-Koblenz)
* `site-ffmo <https://github.com/ffruhr/site-ffmo>`_ (Moers)
* `site-ffmg <https://github.com/ffruhr/site-ffmg>`_ (Mönchengladbach)
* `site-ffm <https://github.com/freifunkMUC/site-ffm>`_ (München)
* `site-ffhmue <https://github.com/Freifunk-Muenden/site-conf>`_ (Münden)
* `site-ffms <https://github.com/FreiFunkMuenster/site-ffms>`_ (Münsterland)
* `site-neuss <https://github.com/ffne/site-neuss>`_ (Neuss)
* `site-ffniers <https://github.com/ffruhr/site-ffniers>`_ (Niersufer)
* `site-ffnw <https://git.nordwest.freifunk.net/ffnw-firmware/siteconf/tree/master>`_ (Nordwest)
* `site-ffrgb <https://github.com/ffrgb/site-ffrgb>`_ (Regensburg)
* `site-ffrn <https://github.com/Freifunk-Rhein-Neckar/site-ffrn>`_ (Rhein-Neckar)
* `site-ffruhr <https://github.com/ffruhr?utf8=✓&query=site>`_ (Ruhrgebiet, Multi-Communities)
* `site-ffs <https://github.com/freifunk-stuttgart/site-ffs>`_ (Stuttgart)
* `site-fftr <https://github.com/freifunktrier/site-fftr>`_ (Trier)
