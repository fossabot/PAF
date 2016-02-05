#PostgreSQL Automatic Failover

High-Availibility for Postgres, based on industry references Pacemaker and
Corosync.

##Description

Pacemaker is nowadays the industry reference for High Availability. In the same
fashion than for Systemd, all Linux distributions moved (or are moving) to this
unique Pacemaker+Corosync stack, removing all other existing high availability
stacks (CMAN, RGManager, OpenAIS, ...). It is able to detect failure on various
services and automatically decide to failover the failing resource to another
node when possible.

To be able to manage a specific service resource, Pacemaker interact with it
through a so-called "Resource Agent". Resource agents must comply to the OCF
specification which define what they must implement (start, stop, promote,
etc), how they should behave and inform Pacemaker of their results.

PostgreSQL Automatic Failover is a new OCF resource Agent dedicated to
PostgreSQL. Its original wish is to keep a clear limit between the Pacemaker
administration and the PostgreSQL one, to keep things simple, documented and
yet powerful.

Once your PostgreSQL cluster built using internal streaming replication, PAF is
able to expose to Pacemaker what is the current status of the PostgreSQL
instance on each node: master, slave, stopped, catching up, etc. Should a
failure occurs on the master, Pacemaker will try to recover it by default.
Should the failure be non-recoverable, PAF allows the slaves to be able to
elect the best of them (the closest one to the old master) and promote it as
the new master. All of this thanks to the robust, feature-full and most
importantly experienced project: Pacemaker.

For information about how to install this agent, see `INSTALL.md`.

##Setup and requirements

PAF supports PostgreSQL 9.3 and higher. It has been extensively tested under
CentOS 6 and 7 in various scenario.

PAF has been written to give to the administrator the maximum control
over their PostgreSQL configuration and architecture. Thus, you are 100%
responsible for the master/slave creations and their setup. The agent
will NOT edit your setup. It only requires you to follow these pre-requisites:

  * slave __must__ be in hot_standby (accept read-only connections)
  * you __must__ provide a template file on each node which will be copied as
    the local `recovery.conf` when needed by the agent
  * the recovery template file __must__ contain `standby_mode = on`
  * the recovery template file __must__ contain `recovery_target_timeline = 'latest'`
  * the `primary_conninfo` parameter in the recovery template file __must__
    set the `application_name` to the node name as seen in Pacemaker
    (usually, the hostname)

When setting up the resource in Pacemaker, here are the available parameters you
can set:

  * `bindir`: location of the PostgreSQL binaries (default: `/usr/bin`)
  * `pgdata`: location of the PGDATA of your instance (default:
    `/var/lib/pgsql/data`)
  * `pghost`: the socket directory or IP address to use to connect to the
    local instance (default: `/tmp`)
  * `pgport`:  the port to connect to the local instance (default: `5432`)
  * `recovery_tpl`: the local template that will be copied as the
    `PGDATA/recovery.conf` file. This template file must exists on all node
    (default: `$PGDATA/recovery.conf.pcmk`)
  * `system_user`: the system owner of your instance's process (default:
    `postgres`)

For a demonstration about how to setup a cluster, see
(http://dalibo.github.com/PAF/documentation.html)[http://dalibo.github.com/PAF/documentation.html].
