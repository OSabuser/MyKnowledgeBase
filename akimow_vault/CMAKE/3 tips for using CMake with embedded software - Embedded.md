#CMAKE #MCU
Vendor-supplied IDEs will generate makefiles for you, but if you want to leverage modern development practices, you need a tool that can help simplify your builds across multiple platforms.

If you’ve been developing embedded software for a while, you know that makefiles are one of the most used tools in our industry. Many vendor-supplied IDEs will generate makefiles for you, so you don’t have to worry about writing them yourself. The problem is that if you want to leverage modern development practices like TDD, DevOps, and CI/CD, you need to write your own or use a tool that can help simplify your builds across multiple platforms.

One tool that has proven its efficiency and versatility in the software industry is CMake. CMake is an open-source, cross-platform family of tools meticulously designed to build, test, and package software. It empowers you to control the software compilation process using simple platform—and compiler-independent configuration files. It generates native makefiles that can be seamlessly used in the compiler environment of your choice.

Let’s look at a few quick tips for using CMake with Embedded Software.

## **Tip #1 – Use toolchain files to simplify build configuration**

Whenever I talk to teams about setting up their build environment, I mention that there are five types of builds they should be supporting:

1. Debug On-Target
2. Release On-Target
3. Simulation
4. Test
5. Code Analysis

Each build type requires the build system to be configured slightly differently. Cross-compilation is necessary for on-target builds. Building for the host and swapping out low-level implementations is essential for simulation. You need to bring in the test harness(es) for testing.

To manage these complexities, you can leverage CMakes’ toolchain files. For example, you might have a toolchain host.cmake file for configuring your simulation environment. The file could be as simple as below:

```
# Use the system's default C and C++ compilers
set(CMAKE_SYSTEM_NAME Darwin) # Adjust this as necessary (e.g., Windows, Darwin for macOS, Linux)
 
# Optionally specify the compiler (GCC/Clang) if the default selection needs to be overridden
set(CMAKE_C_COMPILER gcc)
set(CMAKE_CXX_COMPILER g++)
 
# Add any simulation-specific preprocessor definitions
add_compile_definitions(SIMULATION)
 
# Specify any host-specific compiler flags if necessary
set(COMMON_FLAGS "-O2 -g")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${COMMON_FLAGS}" CACHE STRING "" FORCE)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${COMMON_FLAGS}" CACHE STRING "" FORCE)
 
# Set the C standard and C++ standard that your project requires
set(CMAKE_C_STANDARD 17)
set(CMAKE_CXX_STANDARD 17)
```

You might also create separate toolchain files for each of the other builds. However, I’ve generally found that I can get away with just two: one for host builds and the other for on-target builds.

## **Tip #2 – Use a custom script to invoke CMake builds**

A key to successfully developing embedded software is to automate as much of your development cycle as possible. That includes writing scripts to simplify building your software. CMake commands can get complicated. For example, look at this command for compiling for simulation:

```
cmake -DCMAKE_TOOLCHAIN_FILE=config/toolchain-host.cmake -G Ninja -B build/release -S. -DCMAKE_BUILD_TYPE=Simulation
```

Yes, I can use the up arrow to execute my last command, but no one really wants to memorize that!

Instead, write a script that can be executed with the desired operation. For example, I use a project.sh file that includes all the build types I mentioned before and operations to clean up and execute other utility programs that are helpful during development.

In my script, that long command I would have to remember turns into the following:

```
cmake -DCMAKE_TOOLCHAIN_FILE=$TOOLCHAIN_FILE -G Ninja -B $BUILD_DIR -S. -DCMAKE_BUILD_TYPE=$BUILD_TYPE
```

The parameters I feed to project.sh select the toolchain files, build type, and other useful parameters so that I don’t have to remember them. (It also sets them up for use in a CI/CD system).

## **Tip #3 – Pair CMake with Ninja for faster builds**

You might have noticed in the last tip that the commands I used have Ninja in them. Pairing CMake with Ninja is a great way to help speed up your builds.

Ninja is a small, high-speed build system focused on efficiency. It was created by Evan Martin, a Google engineer, to handle Google’s complex and large builds more effectively than existing build systems. Ninja aims to be much faster than traditional build tools like Make by doing the minimum amount of work necessary to keep files up to date.

In my experience, swapping Ninja for Make has resulted in four to six times faster compilations! Now, you might say, “Who cares!” but if you are following the best practice of static, compile-time checking to save run-time performance, you need to build faster! Even if you aren’t, you likely compile code in rapid succession. Faster builds will limit “wait times” and improve efficiency and throughput.

Using Ninja with CMake is really easy. First, I typically install it in my Docker container. The Dockerfile code is as simple as:

```
RUN apt-get update -y && \\
    apt-get install -y --no-install-recommends \\
    cmake \\
    ninja-build && \\
    apt-get clean && \\
    rm -rf /var/lib/apt/lists/*
```

(Note: There are other ways to force specific versions of CMake and Ninja, but those are beyond the scope of this blog. You can Google it or ask an AI!).

The general form for using CMake to build for Ninja is:

```
cmake -G Ninja -B build
```

We saw earlier that this can vary a bit if you add toolchain files and other definitions. The -G is telling CMake which generator to use. There is a wide range of options, such as:

- Green Hills MULTI = Generates Green Hills MULTI files (experimental, work-in-progress).
- \* Unix Makefiles = Generates standard UNIX makefiles.
- Ninja = Generates build.ninja files.
- Ninja Multi-Config = Generates build-.ninja files.
- Watcom WMake = Generates Watcom WMake makefiles.
- CodeBlocks – Ninja = Generates CodeBlocks project files.
- CodeBlocks – Unix Makefiles = Generates CodeBlocks project files.
- CodeLite – Ninja = Generates CodeLite project files.
- CodeLite – Unix Makefiles = Generates CodeLite project files.
- Eclipse CDT4 – Ninja = Generates Eclipse CDT 4.0 project files.
- Eclipse CDT4 – Unix Makefiles= Generates Eclipse CDT 4.0 project files.
- Kate – Ninja = Generates Kate project files.
- Kate – Unix Makefiles = Generates Kate project files.
- Sublime Text 2 – Ninja = Generates Sublime Text 2 project files.
- Sublime Text 2 – Unix Makefiles = Generates Sublime Text 2 project files.

Once you CMake generates the files for you, all you have to do is run the command:

```
ninja -C build
```

You’ll then see how much faster your code can actually build with the right tools!

## Bottom Line

As the landscape of embedded software development continues to evolve, embracing modern tools and practices becomes increasingly crucial. With its robust and flexible build system, CMake stands out as an indispensable tool for developers aiming to streamline their workflow, improve build times, and facilitate cross-platform development. By leveraging toolchain files, custom build scripts, and efficient build systems like Ninja, you can significantly enhance your development process, enabling you to focus more on innovation and less on the intricacies of build management.

These strategies simplify the complexities of managing different build configurations and align with modern practices such as TDD, DevOps, and CI/CD. As you continue to refine your embedded software development practices, remember that the right tools and techniques can transform challenges into opportunities for greater efficiency and productivity.

---

![](https://www.embedded.com/wp-content/uploads/sites/2/contenteetimes-images-design-embedded-author-beningo290x249.jpg) ***Jacob Beningo*** *is an embedded software consultant who specializes in real-time, microcontroller-based systems. He actively promotes software best practices through numerous articles, blogs, and webinars on topics from software architecture design, embedded DevOps, and implementation techniques. Jacob has 20 years of experience in the field and holds three degrees including a Masters of Engineering from the University of Michigan.*

---

**Related Contents:**

- [3 reasons to ditch Make for CMake:](https://www.embedded.com/3-reasons-to-ditch-make-for-cmake/)
- [Creating a makefile build system using ChatGPT: Makefile simulation target:](https://www.embedded.com/creating-a-makefile-build-system-using-chatgpt-makefile-simulation-target/)
- [5 reasons to build your own C/C++ environment:](https://www.embedded.com/5-reasons-to-build-your-own-c-c-environment/)