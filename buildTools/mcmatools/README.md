# mcmatools
A base layer containing the basic build helper tools
- Make
- CMake
- Automake

## Changelog
- mcmatools:19.01-r1
  - Alpine v3.8
  - automake v1.16.1-r0
  - cmake v3.11.1-r2
  - make=4.2.1-r2

# Usage
`docker run bizeps/mcmatools:latest` to display the version of the installed tools

`docker run bizeps/mcmatools:latest <command to execute>` Example: `docker run bizeps/mcmatools:latest make --version`