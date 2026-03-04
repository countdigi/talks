class: top

## 2026 Bioinformatics Colloquium

*An Overview of Application Container and Workflow Tools Used in TEDDY Bioinformatics Analyses.*

<hr/>

Kevin Counts<br/>
Health Informatics Institute<br/>
University of South Florida<br/>

<hr/>

All slides and resources available at:<br/>
<https://github.com/countdigi/2603-bio-colloq>

---

*Application container and workflow tools...*

--

<hr/>

![apptainer-logo](image/apptainer-logo.png)

--

Builds software containers as isolated, portable, and reproducible units of execution.

--

<hr/>

![snakemake-logo](image/snakemake-logo.png)

--

Executes computational workflows as portable, scalable, and reproducible pipelines.


---

![apptainer logo](image/apptainer-logo.png)

Builds software containers as isolated, portable, and reproducible units of execution.

--

What is a container?

--

*Lightweight, portable, self-contained unit that bundles application code with all its dependencies.*
--

- Shares host OS kernel but otherwise isolated
--

- Native hardware speed (not a VM - no hypervisor)
--

- Behaves consistently across computing environments

---

![apptainer logo](image/apptainer-logo.png)

Why use containers rather than...

--

Traditional software builds
--

- All packages share same OS environment with each other<br/>
  (e.g. `/usr/bin/`, `/usr/lib`, `/usr/local/lib`, ...)
--

- First package built successful<br/>
  `foo` requires `lib-xyz v1`
--

- Next package has conflicting dependencies<br/>
  `bar` requires `lib-xyz v2` breaking `foo`

---

![apptainer logo](image/apptainer-logo.png)

Why use containers rather than...

--

Namespaced software builds
--

- Build `/opt/foo`, `/opt/bar`, ...
--

- Dependencies must be passed with custom flags
  ```bash
  CFLAGS="-I/opt/{foo,bar}/include" \
  LDLAGS="-L/opt/{foo,bar}/lib -Wl,-rpath,/opt/{foo,bar}/lib" \
  ./configure --prefix=/opt/baz
  ```
--

- Difficult/awkward to implement (edit build configs)
--

- Linux distribution [NixOS](https://nixos.org/) embraces this method

---

![apptainer logo](image/apptainer-logo.png)

Why use containers rather than...

--

Conda
--

- Only useful for a subset of tools (Python, R, Julia)
--

- Virtual envs use the same namespace approach<br/>
  (`envs/foo`, `envs/bar`)
--

- Cumbersome copying to other environments (50000+ files)
--

- `environment.yaml` lightweight but identical build?
--

- *Good news*: conda can be built inside a container

---

![apptainer logo](image/apptainer-logo.png)

Containers give each package an isolated namespace so it is always the <b/>only application</b>.

--

<center>
<img src="image/apptainer-container-arch.webp" width="80%" height="80%" />
</center>

---

![apptainer logo](image/apptainer-logo.png)

*Apptainer is an open-source container platform developed for scientific computing / HPC environments*

--

- Originally named "Singularity"
--

- Containers stored as a single `.sif` (Singularity Image File)
--

- Runs as a normal user with no special permissions
--

- Immutable (r/w paths may be mounted inside)
--

- Runs at native hardware speed

---

![apptainer logo](image/apptainer-logo.png)

Apptainer containers:
--

- Isolated filesystem with OS files, libraries, applications, etc.
--

- Base is any version of Linux (`redhat`, `ubuntu`, `debian`)
--

- Base "scratch" for small static binary containers (e.g. rust)
--

- No kernel (uses host kernel machine-language interface)
--

- Linux kernel: Application Binary Interface (ABI)
--

- Must build/run on same arch (`x86_64 != arm64`)

---

![apptainer logo](image/apptainer-logo.png)

<img src="image/apptainer-sif.png" width="100%" height="100%" />

---

![apptainer logo](image/apptainer-logo.png)

Apptainer [definition](https://apptainer.org/docs/user/latest/definition_files.html) file: `spec/debian.def`:
--

```bash
Bootstrap: docker  # docker, localimage, or scratch
From: debian:13.3
```
--

```bash
%post
  apt-get update
  apt-get -y upgrade
  apt-get -y install \
    build-essential \
    ca-certificates \
    curl
```
--

```bash
$ apptainer build tmp/img/debian/13.sif spec/debian.def
#                 output_image          input_definition
```

---

![apptainer logo](image/apptainer-logo.png)

Apptainer definition file `spec/samtools.def`:
--

```bash
Bootstrap: localimage
From: tmp/img/debian/13.3.sif # built in prior example
```
--

```bash
%arguments
  version=1.22
```
--

```bash
%post
  curl -sSL https://github.com/samtools-{{version}}.tar.bz2 \
  | tar jxf -

  cd samtools-{{version}}
  make -j $(nproc)
  make test
  make install # into isolated /usr/local/bin
```

---

![apptainer logo](image/apptainer-logo.png)

Command to build `spec/samtools.def`:
```bash
$ apptainer build \
  --build-arg="version=1.23" \ # {{version}} becomes '1.23'
  tmp/img/samtools-1.23.sif  \ # output_image
  spec/samtools.def            # input_definition
```

---

![apptainer logo](image/apptainer-logo.png)

Executing a container:
```bash
$ apptainer exec tmp/img/samtools-1.23.sif samtools --version
#                container_image           program  options...
```
--

```bash
samtools 1.23
Using htslib 1.23
Copyright (C) 2025 Genome Research Ltd.
... etc.
```

---

![apptainer logo](image/apptainer-logo.png)

Input/Output
--


Your `$HOME` is automatically mounted in container

--

To mount additional paths inside add `--bind` args:
```bash
--bind <host_path>:<container_path>
```
--

E.g.:
```bash
$ apptainer exec --bind=/labdata:/labdata samtools.sif samtools
```

---

![apptainer logo](image/apptainer-logo.png)

Caveats:
--

- Immutable
--

  - Any change requires new container build
--

  - Includes adding/updating versions/libraries
--

- Apptainer inside apptainer
--

  - No experience implementing
--

  - Prefer to build tightly-coupled apps in same container
--

  - Possible? bind-mounting apptainer inside from host

---

![apptainer logo](image/apptainer-logo.png)

In summary...

--

*Builds software containers as isolated, portable, and reproducible units of execution.*
--

- Isolated
--

  - Prevents conflicts in application dependencies
--

- Portable
--

  - Single `sif` file distributed to multiple environments
--

- Reproducible
--

  - Immutable, consistent execution across environments

---

![apptainer logo](image/apptainer-logo.png)

Questions/Comments?

<hr/>

Kevin Counts<br/>
Health Informatics Institute<br/>
University of South Florida<br/>

<hr/>

All slides and resources available at:<br/>
<https://github.com/countdigi/2603-bio-colloq>

---

![snakemake-logo](image/snakemake-logo.png)

--

*Executes computational workflows as portable, scalable, and reproducible pipelines.*

--

- Workflows defined in Python-based language
--

- Builds graph of inputs/outputs and manages execution
--

- Runs steps in-parallel when possible
--

- Runs in local, HPC, or cloud environments

---

![snakemake-logo](image/snakemake-logo.png)

- Snakemake reads a `Snakefile`
--

- `Snakefile`:
--

  - Defines a set of rules
--

  - Rules have `inputs`, `outputs`, `actions`
--

- One or more rules selected as a `target`
--

- Snakemake builds/executes graph of rules to satisfy target
--

- Think "Pull" vs "Push"
--

- Ask for a result, rules to get that result run as needed

---

![snakemake-logo](image/snakemake-logo.png)

```python
# Snakefile

rule all:  # target rule (first rule by default)
    input: "tmp/wrk/filter/a.txt"
```
--

```python
rule select:
    input: "data/a.csv"
    output: "tmp/wrk/select/a.txt"
    shell: "cut -f1,2 {input} > {output}"
```
--

```python
rule filter:
    input: "tmp/wrk/select/a.txt"
    output: "tmp/wrk/filter/a.txt"
    shell: "grep interesting {input} > {output}"
```

---

![snakemake-logo](image/snakemake-logo.png)

DAG (Directed Acyclic Graph) w/ `bcftools_call` as target

--

`Directed`: One-way dependencies / `Acyclic`: No loops

--

![dag](image/dag_call.webp)

---

![snakemake-logo](image/snakemake-logo.png)

A rule is executed if:
--

- Output is input of rule in graph satisfying the target
--


Once all rules are run, rules will not run again unless:
--

- An input is newer than the output (cascades...)
--

- An input is updated
--

- The code in a rule changes (action section, e.g. `shell:`)

--

Snakemake uses timestamps/hashes to track these changes.

---

![snakemake-logo](image/snakemake-logo.png)

Wildcards - adding samples "b" and "c"
--

```python
rule all:
    input:
       "tmp/wrk/filter/a.txt",
       "tmp/wrk/filter/b.txt",
       "tmp/wrk/filter/c.txt"
```
--

```bash
rule select:
    input: "data/{sample}.csv"
    output: "tmp/wrk/select/{sample}.txt"
    shell: "cut -f1,2 {input} > {output}"

rule filter:
    input: "tmp/wrk/select/{sample}.txt"
    output: "tmp/wrk/filter/{sample}.txt"
    shell: "grep interesting {input} > {output}"
```

---

![snakemake-logo](image/snakemake-logo.png)

Wildcards w/ `expand` helper function
--

```python
SAMPLES = ["a", "b", "c"]
```
--

```python
rule all:
    input:
       expand("tmp/wrk/filter/{sample}.txt", sample=SAMPLES)
```
--

```python
rule select:
    input: "data/{sample}.csv"
    output: "tmp/wrk/select/{sample}.txt"
    shell: "cut -f1,2 {input} > {output}"

rule filter:
    input: "tmp/wrk/select/{sample}.txt"
    output: "tmp/wrk/filter/{sample}.txt"
    shell: "grep interesting {input} > {output}"
```

---


![snakemake-logo](image/snakemake-logo.png)

Command-line interface

```bash
snakemake
```
--

```bash
snakemake --snakefile=Snakefile
```
--

```bash
snakemake --dry-run
```
--

```bash
snakemake --dag | dot -Tsvg > DAG.svg
```

---

![snakemake-logo](image/snakemake-logo.png)

[bin/install-snakemake](https://github.com/countdigi/2603-bio-colloq/blob/main/bin/install-snakemake)
--

- Downloads miniforge
--

- Creates `$HOME/opt/snake/<version>`
--

- Creates virtual environment under `envs/main`
--

- Install packages in `envs/main`:
--

  - `snakemake`
--

  - `snakemake-executor-plugin-slurm`

--

Add to your `$HOME/.bashrc`:
--

```bash
export PATH=${HOME}/opt/snake/9.16.3/envs/main/bin:${PATH}
```

---

![snakemake-logo](image/snakemake-logo.png)

TEDDY GWAS Imputation example:
- <a href="image/gwas-dag.svg">DAG</a>
- <a href="Snakefile.html">Snakefile</a>

---

![snakemake-logo](image/snakemake-logo.png)

STAMPEDE head-node: `alpha1.epi.usf.edu`

--

`${HOME}` is set to `${HPC_HOME}`

--


`${HPC_HOME}` (HPC scratch / Not backed-up)
--

- `/stampede1/user/<user>` (e.g. `/stampede1/user/countskm`)

--

`${NET_HOME}`
--

- `/home/<user>` (e.g.: `/home/countskm`)

---

![snakemake-logo](image/snakemake-logo.png)

Use profile to enable [slurm-executor](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html) plugin:

--

```bash
$ snakemake --profile=profiles/stampede
```
--

```bash
$ cat profiles/stampede/config.yaml

executor: slurm
default-resources:
  slurm_account: epi
  slurm_partition: stampede01
  runtime: 4h
  mem: 10G
jobs: 100
max-jobs-per-second: 1
```

---

![snakemake-logo](image/snakemake-logo.png)

Questions/Comments?

<hr/>

Kevin Counts<br/>
Health Informatics Institute<br/>
University of South Florida<br/>

<hr/>

All slides and resources available at:<br/>
<https://github.com/countdigi/2603-bio-colloq>
