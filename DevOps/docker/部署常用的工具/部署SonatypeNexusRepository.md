```
[
    {
        "Id": "26dc69aa341043811d87db22ad7ab0a81fa9e084286b2469e8e8c153cc4e17ca",
        "Created": "2025-10-09T07:11:20.094546473Z",
        "Path": "/opt/sonatype/nexus/bin/nexus",
        "Args": [
            "run"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 3354,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2026-02-24T01:17:04.291104858Z",
            "FinishedAt": "2026-02-10T07:13:57.938971647Z"
        },
        "Image": "sha256:d3df1178e8d739d9dad1fe796aee8b9d19789b850289e2b01c9edc587f4464ad",
        "ResolvConfPath": "/data/docker-data/default/containers/26dc69aa341043811d87db22ad7ab0a81fa9e084286b2469e8e8c153cc4e17ca/resolv.conf",
        "HostnamePath": "/data/docker-data/default/containers/26dc69aa341043811d87db22ad7ab0a81fa9e084286b2469e8e8c153cc4e17ca/hostname",
        "HostsPath": "/data/docker-data/default/containers/26dc69aa341043811d87db22ad7ab0a81fa9e084286b2469e8e8c153cc4e17ca/hosts",
        "LogPath": "/data/docker-data/default/containers/26dc69aa341043811d87db22ad7ab0a81fa9e084286b2469e8e8c153cc4e17ca/26dc69aa341043811d87db22ad7ab0a81fa9e084286b2469e8e8c153cc4e17ca-json.log",
        "Name": "/nexus3",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "unconfined",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "/etc/localtime:/etc/localtime:ro",
                "/data/application/nexus3:/nexus-data"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {
                    "max-file": "3",
                    "max-size": "100m"
                }
            },
            "NetworkMode": "bridge",
            "PortBindings": {
                "8081/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8081"
                    }
                ],
                "8082/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8082"
                    }
                ],
                "8083/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8083"
                    }
                ],
                "8084/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8084"
                    }
                ],
                "8085/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8085"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "always",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "ConsoleSize": [
                41,
                304
            ],
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": true,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": [
                "label=disable"
            ],
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": [],
            "BlkioDeviceWriteBps": [],
            "BlkioDeviceReadIOps": [],
            "BlkioDeviceWriteIOps": [],
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": [],
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": null,
            "ReadonlyPaths": null
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/data/docker-data/default/overlay2/b993dcb56b06fe4384ede082ff66ff71661fd7111f5df5e6f4d1148128b4e9ab-init/diff:/data/docker-data/default/overlay2/fdf148dca13f8b93851dcaf1b8e193e733569a9402ec39206ff82e315f1c55f4/diff:/data/docker-data/default/overlay2/8e1d8c1bb9907bc033f1fe8be3cb12841bdb99ab5922f60844d451a666a5bb41/diff:/data/docker-data/default/overlay2/28f06419e3292b174e4b2c5a0d5f740c992e86f15cf396735ffd3fd1c041ca23/diff:/data/docker-data/default/overlay2/ed013b2bf39a8d19ada8dfd2fdae2035100a484082e7a1a10fcf169f060c931a/diff:/data/docker-data/default/overlay2/a25d61da8f8eb56b24e750b436d4658992b9de8cfbcd447dfa8c7a8c5b15b840/diff:/data/docker-data/default/overlay2/8938f006daf27345d05d1822f2f69a725e2930b94729d8383ad35b813f0cb562/diff:/data/docker-data/default/overlay2/157103b3c92d449472db0f976972f07f11e2f11985de5b590433158a1e077989/diff",
                "MergedDir": "/data/docker-data/default/overlay2/b993dcb56b06fe4384ede082ff66ff71661fd7111f5df5e6f4d1148128b4e9ab/merged",
                "UpperDir": "/data/docker-data/default/overlay2/b993dcb56b06fe4384ede082ff66ff71661fd7111f5df5e6f4d1148128b4e9ab/diff",
                "WorkDir": "/data/docker-data/default/overlay2/b993dcb56b06fe4384ede082ff66ff71661fd7111f5df5e6f4d1148128b4e9ab/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/etc/localtime",
                "Destination": "/etc/localtime",
                "Mode": "ro",
                "RW": false,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/data/application/nexus3",
                "Destination": "/nexus-data",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
        "Config": {
            "Hostname": "26dc69aa3410",
            "Domainname": "",
            "User": "nexus",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "8081/tcp": {},
                "8082/tcp": {},
                "8083/tcp": {},
                "8084/tcp": {},
                "8085/tcp": {}
            },
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": false,
            "Env": [
                "INSTALL4J_ADD_VM_PARAMS=-Xms512M -Xmx512M -XX:MaxDirectMemorySize=1g",
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "container=oci",
                "SONATYPE_DIR=/opt/sonatype",
                "NEXUS_HOME=/opt/sonatype/nexus",
                "NEXUS_DATA=/nexus-data",
                "NEXUS_CONTEXT=",
                "SONATYPE_WORK=/opt/sonatype/sonatype-work",
                "DOCKER_TYPE=rh-docker"
            ],
            "Cmd": [
                "/opt/sonatype/nexus/bin/nexus",
                "run"
            ],
            "Image": "sonatype/nexus3:3.75.1",
            "Volumes": {
                "/nexus-data": {}
            },
            "WorkingDir": "/opt/sonatype",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "architecture": "x86_64",
                "base-image-ref": "https://catalog.redhat.com/software/containers/ubi8/ubi-minimal/5c359a62bed8bd75a2c3fba8?architecture=amd64&image=6722ca74023a282abf2b7f50",
                "build-date": "2024-10-30T23:56:36",
                "com.redhat.component": "ubi8-minimal-container",
                "com.redhat.license_terms": "https://www.redhat.com/en/about/red-hat-end-user-license-agreements#UBI",
                "com.sonatype.license": "Apache License, Version 2.0",
                "com.sonatype.name": "Nexus Repository Manager base image",
                "description": "The Nexus Repository Manager server           with universal support for popular component formats.",
                "distribution-scope": "public",
                "io.buildah.version": "1.33.8",
                "io.k8s.description": "The Nexus Repository Manager server           with universal support for popular component formats.",
                "io.k8s.display-name": "Nexus Repository Manager",
                "io.openshift.expose-services": "8081:8081",
                "io.openshift.tags": "Sonatype,Nexus,Repository Manager",
                "maintainer": "Sonatype <support@sonatype.com>",
                "name": "Nexus Repository Manager",
                "release": "3.75.1",
                "run": "docker run -d --name NAME           -p 8081:8081           IMAGE",
                "stop": "docker stop NAME",
                "summary": "The Nexus Repository Manager server           with universal support for popular component formats.",
                "url": "https://sonatype.com",
                "vcs-ref": "4f8da2b64a13f2a264bd802d8909bf803211fb20",
                "vcs-type": "git",
                "vendor": "Sonatype",
                "version": "3.75.1-01"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "fa8a81b46448a460d36a4968e6fe461ec64d88b3e96c9e721ba363eb68c95ede",
            "SandboxKey": "/var/run/docker/netns/fa8a81b46448",
            "Ports": {
                "8081/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8081"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "8081"
                    }
                ],
                "8082/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8082"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "8082"
                    }
                ],
                "8083/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8083"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "8083"
                    }
                ],
                "8084/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8084"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "8084"
                    }
                ],
                "8085/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8085"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "8085"
                    }
                ]
            },
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "4ca84012987d1dd1aab57316c3f79db016a621bf3f5d674047061bf2ef515547",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.3",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:03",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "02:42:ac:11:00:03",
                    "DriverOpts": null,
                    "NetworkID": "2afb6fab1fc319b530604ed715029ce21e5830dc1ef4f889b22647bb449b4284",
                    "EndpointID": "4ca84012987d1dd1aab57316c3f79db016a621bf3f5d674047061bf2ef515547",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": null
                }
            }
        }
    }
]

```

