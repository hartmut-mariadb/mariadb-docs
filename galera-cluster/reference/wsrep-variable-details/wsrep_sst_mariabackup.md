# wsrep\_sst\_mariabackup

#### `wsrep_sst_mariabackup` Variables

The `wsrep_sst_mariabackup` script handles the actual data transfer and processing during an SST. The variables it reads from the `[sst]` group (and sometimes from other groups, too) control aspects of the backup format, compression, transfer mechanism, and logging.

The `wsrep_sst_mariadbbackup` script parses the following options:

* `streamfmt` (`[sst]`)
  * Default: `mbstream`
  * Description: Defines the streaming format used by `mariabackup` for the SST. `mbstream` indicates that `mariabackup` will output a continuous stream of data. `xbstream` is still supported as an alias for `mbstream` for backwards compatibility reasons. `tar` is the only other valid format, but is rarely used as it is less flexible than `mbstream`; missing support for some features.

***

* `transferfmt` (`[sst]`)
  * Default: `socat`
  * Description: Specifies the transfer format or utility used to move the data stream from the donor to the joiner node. `socat` is a common command-line tool for data transfer, often used for setting up various network connections, but its predecessor `netcat` can also still be used by setting this parameter to `nc` instead of `socat`.

***

* `sockopt` (`[sst]`)
  * Description: Allows additional socket options to be passed to the underlying network communication. This could include settings for TCP buffers, keep-alives, or other network-related tunables to optimize the transfer performance. Possible values are basically all valid command line options of the `socat` or `netcat` tool, whichever was chosen by the `transferfmt` setting

***

* `progress` (`[sst]`)
  * Default: `0`
  * Description: When having this set to `1`, and the `pv` tool installed, extra progress information will be written to the error log showing how transfer of the data progresses.
  
***

* `time` (`[sst]`) 
 * Default: `0`
 * Description: When set to `1` extra timeing info will be printed for each backup stage in the form `NOTE: _stage_name_ took ### seconds`f
 
***

* `cpat` (`[sst]`)
  * Default: `'.*\.pem$\|.*galera\.cache$\|.*sst_in_progress$\|.*\.sst$\|.*gvwstate\.dat$\|.*grastate\.dat$\|.*\.err$\|.*\.log$\|.*RPM_UPGRADE_MARKER$\|.*RPM_UPGRADE_HISTORY$'`
  * Description: Before starting the actual data transfer several non-database files will be removed from the joiners datadir. The `cpat` pattern is a regular expression acting as a positive list of file name patters to be kept, not deleted.

***

* `compressor` (`[sst]`)
  * Description: Specifies the compression utility to be used on the data stream before transfer. Common values could include `gzip`, `pigz`, `lz4`, or 'bzip2'  which reduce the data size for faster transmission over the network. Output will be piped through the given command, additiona command line options can be given as needed, e.g. `pigz --fast` or `pigz --best` to control compression rate.

***

* `decompressor` (`[sst]`)
  * Description: Specifies the decompression utility to be used on the receiving end (joiner node) to decompress the data stream that was compressed by `compress`. It should correspond to the `compress` setting, e.g. when using `compresor=pigz` then `decompressor=unpigz` should be used for decompression to work.

***

* `rlimit` (resource limit)
  * Description: Potentially sets resource limits for the `mariabackup` process during the SST. This could include limits on CPU usage, memory, or file descriptors, preventing the SST from consuming excessive resources and impacting the server's performance.

***

* `uextra` (use-extra)
  * Default: `0`
  * Description: A boolean flag (0 or 1) that likely indicates whether to use extra or advanced features/parameters during the SST. The specific "extra" features would be determined by the `mariabackup` implementation.

***

* `speciald` (sst-special-dirs)
  * Default: `1`
  * Description: A boolean flag (0 or 1) that likely controls whether `mariabackup` should handle special directories (e.g., `innodb_log_group_home_dir`, `datadir`) in a specific way during the SST, rather than just copying them as regular files. This is important for maintaining data consistency.

***

* `stimeout` (sst-initial-timeout)
  * Default: `300`
  * Description: Sets an initial timeout in seconds for the SST process. If the SST doesn't make progress or complete within this initial period, it might be aborted.

***

* `ssyslog` (sst-syslog)
  * Default: `0`
  * Description: A boolean flag (0 or 1) that likely controls whether SST-related messages should be logged to syslog. This can be useful for centralized logging and monitoring of Galera cluster events.

***

* `sstlogarchive` (sst-log-archive)
  * Default: `1`
  * Description: A boolean flag (0 or 1) that likely determines whether SST logs should be archived. Archiving logs helps in post-mortem analysis and troubleshooting of SST failures.

***

* `sstlogarchivedir` (sst-log-archive-dir)
  * Description: Specifies the directory where SST logs should be archived if `sstlogarchive` is enabled.

