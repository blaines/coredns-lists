# lists

## Name

*lists* - enables whitelists and blacklists for CoreDNS

## Description

The *lists* plugin is used to filter DNS requests based on lists of zones. It can be configured both as a *whitelist* and as a *blacklist*. It's default behavior is strict with the whitelist and the blacklist empty (no zones permitted, no zones blocked). If a *whitelist* is configured in addition to a *blacklist*, __the *blacklist* always takes precedence__! New or changed zone *files* are automatically picked up from disk. The *lists* plugin should be followed by an authoritive source of DNS records (i.e. `forward`).

## Syntax

~~~
lists {
    whitelist {
        zones {
            [ZONES...]
        }
        files {
            [FILES...]
        }
    }
    blacklist {
        zones {
            [ZONES...]
        }
        files {
            [FILES...]
        }
    }
    dryrun
}
~~~


* `whitelist` - Contains the configuration for permitted zones. Lists are composed of `zones` and `files`. If `whitelist` is empty, no zones are permitted.
* `blacklist` - Contains the configuration for blocked zones. Lists are composed of `zones` and `files`. If `blacklist` is empty, no zones are blocked.
* `zones` - A list of **ZONES** to be included on the list. The default is empty. Each zone should end with a period.
* `files` - A list of **FILES** containing a list of zones to be included on the list. The default is empty. Each zone should end with a period and there should be one zone per line.
* `dryrun` - Execute the plugin without any blocking behavior. Use with the `debug` plugin to see expected behavior. Useful for diagnosing list issues.

Any match in a list for a zone will apply that list's logic. When a zone matches in both the whitelist and blacklist, the blacklist takes precedence. A match is the most specific zone for the query (longest suffix match). E.g. if there are two zones in a list, one for example.org and one for a.example.org, and the query is for www.a.example.org, it will be matched to the latter.

* If example.org is on the whitelist but a.example.org is in the blacklist, and the query is for www.a.example.org, the query will be denied. (a.example.org matches the blacklist)
* If example.org is on the blacklist but a.example.org is in the whitelist, and the query is for www.a.example.org, the query will be denied. (example.org matches the blacklist)
* If example.org and/or a.example.org is in the whitelist, and the query is for www.a.example.org, the query will be permitted.
* If api.example.org is on the blacklist, and `.` is in the whitelist, the query is for www.a.example.org, the query will be permitted. (api.example.org does not match a.example.org)

## Examples
~~~
. {
    lists {
        whitelist {
            zones {
                example.com
            }
            files {
                ./whitelist-zones
            }
        }
        blacklist {
            zones {
                example.com
            }
            files {
                /etc/coredns/blacklist-zones
            }
        }
    }
    forward . 1.1.1.1
    cache 30
}
~~~
