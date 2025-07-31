# wsrep\_sst\_common

#### `wsrep_sst_common` Variables

The `wsrep_sst_common` script provides shared functionality used by various State Snapshot Transfer (SST) methods in Galera Cluster. It centralizes the handling of common configurations such as authentication credentials, `SSL/TLS` encryption parameters, and other security-related settings. This ensures consistent and secure communication between cluster nodes during the SST process.

The `wsrep_sst_common` script parses the following options:

***

* `wsrep-sst-auth` ( `[sst]` )
  * Description: Defines the authentication credentials used by the State Snapshot Transfer (SST) process, typically formatted as `user:password`. These credentials are essential for authenticating the SST user on the donor node, ensuring that only authorized joiner nodes can initiate and receive data during the SST operation. Proper configuration of this variable is critical to maintain the security and integrity of the replication process between Galera cluster nodes.

***

* `tca` ( `[sst]` ) / `ssl_ca` ( `[sst]`, ... )
  * Description: Specifies the Certificate Authority (CA) certificate file used for SSL/TLS encryption during State Snapshot Transfers (SSTs). When encryption is enabled, this certificate allows the joining node (client) to authenticate the identity of the donor node, ensuring secure and trusted communication between them.

***

* `tcap` ( `[sst]` ) / `ssl_capath` ( `[sst]`, ... )
  * Description: Specifies the path to a directory that contains a collection of trusted Certificate Authority (CA) certificates. Instead of providing a single CA certificate file, this option allows the use of multiple CA certificates stored in separate files within the specified directory. It is useful in environments where trust needs to be established with multiple certificate authorities.

***

* `tpem` ( `[sst]` ) / `ssl_cert` ( `[sst]`, ... )
  * Description: This variable stores the path to the TLS/SSL certificate file for the specific node. The certificate, typically in PEM format, is used by the node to authenticate itself to other nodes during secure SST operations. It is derived from the `tcert` option in the `[sst]` section.

***

* `tkey` ( `[sst]` ) / `ssl_key` ( `[sst]`, ... )
  * Description: Represents the private key file that corresponds to the public key certificate specified by `tpem`. This private key is essential for decrypting data and establishing a secure connection during State Snapshot Transfer (SST). It enables the receiving node to authenticate encrypted information and participate in secure replication within the cluster.

The classig Galera SSL/TLS setting names starting with `t...` take precedence over those starting with `ssl...` if both varants are present. The `t...` options are only checked for in the `[sst]` section while the the `ssl...` options may also be ckecked for in SST method specific sections; e.g. the `wsrep_sst_mariabackup` script also looks for `ssl...` options in the `[mariabackup]` and `[xtrabackup]` sections of the configuration file(s).
