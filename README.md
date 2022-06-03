![Code Quality Workflow](https://github.com/ghrcdaac/dmrpp-file-generator-docker/actions/workflows/code-quality.yml/badge.svg?branch=master)
![Code Quality](https://api.codiga.io/project/33592/score/svg)
![Code Grade](https://api.codiga.io/project/33592/status/svg) <br>
Code Quality [Dashboard](https://app.codiga.io/project/33592/dashboard)


# How to install
```
$ pip install git+https://github.com/ghrcdaac/dmrpp-file-generator-docker.git@master#egg=dmrpp-file-generator
```

# dmrpp-file-generator-docker
Docker image to generate dmrpp files from netCDF and HDF files
# Supported get_dmrpp configuration

## Via Cumulus Collection configuration
```code
{
  "config": {
    "meta": {
      "dmrpp": {
        "dmrpp_regex" : "^.*.H6",
        "options": [
          {
            "flag": "-M"
          },
          {
            "flag": "-s",
            "opt": "s3://ghrcsbxw-public/dmrpp_config/file.config",
            "download": "true"
          },
          {
            "flag": "-c",
            "opt": "s3://ghrcsbxw-public/aces1cont__1/aces1cont_2002.212_v2.50.tar.cmr.json",
            "download": "false"
          }
        ]
      }
    }
  }
}
```

`opt` is the value that will come after the flag, if provided with `"download": "true"` the value will be ignored and the file provided will be downloaded and used with the `flag`.
If the `opt` is provided with `"download": "false"` or without `download` the value of `opt` will be used as a letteral string in `get_dmrpp` executable.
We are supporting HTTP and s3 protocols.

## Via Cumulus Workflow configuration

If you want the same `dmrpp` config to apply to multiple collections that use
the same workflow, the configuration can be placed in the workflow at
`${StepName}.Parameters.cma.task_config.dmrpp` instead of in each collection at
`config.meta.dmrpp`:

```
   "HyraxProcessing": {
      "Parameters": {
        "cma": {
          "event.$": "$",
          "task_config": {
            ...
            "dmrpp": {
              "dmrpp_regex" : "^.*.H6",
              "options": [
                {
                  "flag": "-M"
                },
                {
                  "flag": "-s",
                  "opt": "s3://ghrcsbxw-public/dmrpp_config/file.config",
                  "download": "true"
                },
                {
                  "flag": "-c",
                  "opt": "s3://ghrcsbxw-public/aces1cont__1/aces1cont_2002.212_v2.50.tar.cmr.json",
                  "download": "false"
                }
              ]
            }
          }
        }
      },

    ...
    }
```

As with other variables in the JSON template file, the `dmrpp` config can be set in a terraform variable:

```
"task_config": {
  ...
  "dmrpp": ${jsonencode(dmrpp_config)}
}
```

If `dmrpp` configuration is set on the collection and in the workflow, any keys
in the collection's configuration will override those keys in the
workflow's. This means, for example, if you want all collections processed with
a given workflow to use the same `options` but different `dmrpp_regex`, the
workflow config can set `options` and each collection can set `dmrpp_regex` to
get the desired behavior.

# Supported get_dmrpp configuration
## Via env vars
Create a PAYLOAD environment variable holding dmrpp options
```
PAYLOAD='{"dmrpp_regex": "^.*.nc4", "options":[{"flag": "-M"}, {"flag": "-s", "opt": "s3://ghrcsbxw-public/dmrpp_config/file.config","download": "true"}]}'
```
`dmrpp_regex` is optional to override the DMRPP-Generator regex
# Generate DMRpp files locally without Hyrax server
`generate-validate-dmrpp` now uses docker compose v2.  Please update to  
docker compose v2 or you will get the error  
`/bin/sh: 1: docker compose: not found`
```
$generate-validate-dmrpp --help
usage: generate-validate-dmrpp [-h] -p NC_HDF_PATH [-prt PORT] [-pyld PAYLOAD] [-vldt VALIDATE]

Generate and validate DMRPP files.

optional arguments:
  -h, --help            show this help message and exit
  -p NC_HDF_PATH, --path NC_HDF_PATH
                        Path to netCDF4 and HDF5 folder
  -prt PORT, --port PORT
                        Port number to Hyrax local server
  -pyld PAYLOAD, --payload PAYLOAD
                        Payload to execute get_dmrpp binary
  -vldt VALIDATE, --validate VALIDATE
                        Validate netCDF4 and HDF5 files against OPeNDAP local server

```

The folder `<absolute/path/to/nc/hdf/files>` should contain netCDF and/or HDF files
```code
generate-validate-dmrpp -p <absolute/path/to/nc/hdf/files> -vldt false
```
<a href="https://asciinema.org/a/p6xzJQguUni26FIbjCxm8giWw" target="_blank"><img src="https://asciinema.org/a/p6xzJQguUni26FIbjCxm8giWw.svg" /></a>
# Generate DMRpp files locally with Hyrax server (for validation)

```code
generate-validate-dmrpp -p <absolute/path/to/nc/hdf/files>
A prompt will ask you to visit localhost:8080
# If you want to change the default port run the command with
generate-validate-dmrpp -p <absolute/path/to/nc/hdf/files> -prt 8889
Now you can validate the result in localhost:8889
```

<a href="https://asciinema.org/a/1NbdKMckp3ONLAuD1zbDkCFIw" target="_blank"><img src="https://asciinema.org/a/1NbdKMckp3ONLAuD1zbDkCFIw.svg" /></a>

# Generate missing metadata for non-netcdf compliant data (the -b switch)
```code
generate-validate-dmrpp -p <absolute/path/to/nc/hdf/files> -pyld $PAYLOAD
```
or
```code
docker run --rm -it --env-file ./env.list -v <absolute/path/to/nc/hdf/files>:/workstation ghrcdaac/dmrpp-generator
```
where PAYLOAD contains your flags and switches
```code
PAYLOAD={"options":[{"flag": "-M"}, {"flag": "-u", "opt": "/usr/share/hyrax"}]}
```
