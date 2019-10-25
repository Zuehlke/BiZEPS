[home](01_BiZEPS_Introduction.md)

#   Interesting Use Cases

##  Collection of Tools and Chains
- Whenever some one sets up a new tool chain
- Generate a Docker image from a docker file
  - Documentation (with docker file)
  - Reusability
- Available tools and chains will grow over time

##  Tool Chain Isolation
- Each tool chain has its own virtual environment
- No conflicts between different tools and versions

##  Use BiZEPS on Development Machine
- No local installation of the tool chain
  - Just get the images from the Docker repository
  - No difference to the build server tool chain
    - Libraries
    - Toolchain
    - OS
- Use the same containers as the build server uses
- Simplified update of the developer machine tool chains

##  Build System Base Lines (Versions)
- Each Docker image has its release version
- A base line is a collection of images and their versions
- Allows releases of the build system
- Allows to run any build system version at any time

##  Continuous Integration of Build System
- Create a new image with some changes
  - New tool
  - New version
  - ...
- Define a 'latest' build system base line (not yet released)
- Generate the 'latest' images from the 'latest' base line
- Start the 'latest' containers on the server
- Build with the 'latest' build system for test purposes
- Keep the currently released base line running in parallel for your daly business
- If the 'latest' base line is stable: release it

##  Scalability
- Let all containers run on a single server (or development machine)
- Let build slaves run on other servers
- Connect to cloud
  - Break resource peaks
  - Outsource full build infrastructure
