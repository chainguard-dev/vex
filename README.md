# `vexctl`: A tool to make VEX work

`vexctl` is a tool to apply and attest [VEX](https://github.com/openvex/spec/blob/main/OPENVEX-SPEC.md) 
(Vulnerability Exploitability eXchange)cdata. Its purpose is to "turn off" alerts of vulnerabilities that 
do not affec a product.

VEX can be thought of as a "negative security advisory" which software authors can use to 
communicate to their users that a reportedly vulnerable component has no security 
implications for their product.
 
## Operational Model

There are two main modes of operation: 
_attesting_ VEX statements and _filtering_ ("VEXing") security scanner results.

### 1. Attest VEX Statements

VEX documents can be created as a file on disk or captured in a
signed attestation which can be attached to a container image. The data is
generated from a known rule set (the "golden data") which is reapplied for 
new releases of the same project.

#### Generation Examples

```shell
# Attest and attach VEX statements in mydata.vex.json to a container image:
$ vexctl attest --attach --sign mydata.vex.json cgr.dev/image@sha256:e4cf37d568d195b4..
```

### 2. Filter Results

Using the statements in file or from an attestation, `vexctl` 
can filter security scanner results to remove _VEX'ed out_ entries.

#### Filtering Examples

```shell
# From a VEX file:
$ vexctl filter scan_results.sarif.json vex_data.csaf

# From a stored VEX attestation:
$ vexctl filter scan_results.sarif.json cgr.dev/image@sha256:e4cf37d568d195b4b5af4c36a...
```

The output from both examples filters out the results that are not exploitable. We only support 
[SARIF](https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html) results for now, but 
plan to add support for the propietary formats of the most popular scanners.

### Multiple VEX Files

Assessing impact of a vulnerability takes time, so VEX is designed
with this in mind. An example timeline may look like this:

1. A project becomes aware of `CVE-2023-12345`, associated with one of its components.
2. Developers issue a VEX document with a status of [`under_investigation`](https://github.com/openvex/spec/blob/main/OPENVEX-SPEC.md#status-labels) to
inform their users they are aware of the CVE but are checking what impact it has.
3. After investigation, the developers determine the CVE has no impact
in their project because the vulnerable function in the component is never executed.
4. They issue a second VEX document with a status of [`not_affected`](https://github.com/openvex/spec/blob/main/OPENVEX-SPEC.md#status-labels) with the [`vulnerable_code_not_in_execute_path`](https://github.com/openvex/spec/blob/main/OPENVEX-SPEC.md#status-justifications) justification.

`vexctl` will read all the documents in cronological order and "replay" the
known impacts statuses the order they were found, effectively computing the
`not_affected` status.

## Build

To build `vexctl` clone this repository and run simply run `make`.

```console
$ git clone git@github.com:chainguard-dev/vex.git
$ cd vex
$ make
$ ./vexctl version
 _   _  _____ __   __ _____  _____  _
| | | ||  ___|\ \ / //  __ \|_   _|| |
| | | || |__   \ V / | /  \/  | |  | |
| | | ||  __|  /   \ | |      | |  | |
\ \_/ /| |___ / /^\ \| \__/\  | |  | |____
 \___/ \____/ \/   \/ \____/  \_/  \_____/
vexctl: A tool for working with VEX data

GitVersion:    devel
GitCommit:     unknown
GitTreeState:  unknown
BuildDate:     unknown
GoVersion:     go1.19
Compiler:      gc
Platform:      linux/amd64
```
