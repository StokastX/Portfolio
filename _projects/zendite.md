---
title: Zendite Engine (contributor)
layout: single
toc: true
toc_sticky: true
permalink: /zendite/
header:
  teaser: /assets/images/teasers/zendite_engine.jpg
excerpt: A small-scale game engine written in C++ and OpenGL
order: 3
year: 2024
---

<a href="https://github.com/StokastX/ZenditeEngineV2" class=""> <sup><i class="fa-brands fa-github"></i> Github</sup></a>

A small scale 3D game engine written in C++ using OpenGL.

![Capture d'écran 2024-03-21 171205](https://github.com/Patoche692/ZenditeEngineV2/assets/54531293/74387bf7-57b1-49c6-b833-251babb627d5)

![Capture d'écran 2024-03-21 171250](https://github.com/Patoche692/ZenditeEngineV2/assets/54531293/a75cc06a-baca-4fa8-ad0f-0d52ad7b2d70)

![Capture d'écran 2024-03-21 171415](https://github.com/Patoche692/ZenditeEngineV2/assets/54531293/13326740-9506-479d-941b-115fbdc64f01)

## Features

- Entity component system (ECS)
- Model loading system
- Lighting system
  - Point, spot and directional light components
  - Shadow mapping
- UI to let the user manipulate the properties of every entity
- AABB collision detection

## Prerequisites

1.	Use the Windows 10/11 operating system
2.	Ensure you have CMAKE installed and added to the system path on your computer.
3.	Ensure you have Visual Studio 2022 installed with the C++ workload.

## Build

If the above are all as stated, perform the following steps:

1.	When you pulled this repo from git, ensure that you used git clone --recurse-submodules <repository-url>  (otherwise, the assimp submodule will not be included)
2.	In the Solution Directory, navigate to dep/assimp
3.	In the “assimp” directory, open a bash terminal and type “cmake CMakeLists.txt” an push enter.
4.	Then type “cmake --build .” and push enter again. This will set up the assimp submodule.
5.	Now double-click the visual studio solution file in the solution directory “zenditeEngineV2.sln” to open it in Visual Studio 2022.
6.	Press F5 to build and run.

## Usage

Controls:

- W, A, S and D => Move flycam around the scene.
- Q and E => Hover Up and Down.
- C => Unlock mouse pointer (Which enables the user to use the mouse on the GUI).
- V => Lock mouse pointer to game window (Nouse will now control camera view direction).
- L and K => Toggle wireframe mode on and off.

## Dependencies

The following 3rd Party Libs were used in this project:

- GLFW => For window context creation
	Link: https://www.glfw.org/
- GLEW => For retrieving OpenGL functions
	Link: https://glew.sourceforge.net/
- Assimp => for model loading
	Link: https://github.com/assimp/assimp
- Stb image => for texture image loading
	Link: https://github.com/nothings/stb
- glm => for maths operations (mainly for matrix and vector calculations)
	Link: https://github.com/g-truc/glm
- imgui => for the GUI implementation (implemented in menu.h)
	Link: https://github.com/ocornut/imgui



Other 3rd Party Contributions:

- LearnOpengl.com's camera class implementation. (Implemented as Camera.h)
	Links: https://learnopengl.com/Getting-started/Camera and https://learnopengl.com/code_viewer_gh.php?code=includes/learnopengl/camera.h
- The Cherno's Basic OpenGl error checking macro from his OpenGl error checking YouTube video. (Implemented on lines 21 - 24 of utils.h)
	Link: https://www.youtube.com/watch?v=FBbPWSOQ0-w
- Austin Morlan's C++ entity component system blog. I used this design for the overall ECS design. With some modifications, such as having a higher level Coordinator on top of ECSCoordinator.h. (Implemented: All .h files within the src/ECS directory of this repo)
	Link: https://austinmorlan.com/posts/entity_component_system/
- Learnopengl textures => The following textures in the res/textures dir were taken from learnopengl.com: (awesomeface.png, container2.png, container2_specular.png, wall.png)
- res/textures/lava.jpg texture => taken from Maiomi on Pintrest: https://www.pinterest.com/pin/306737424617953515/
- res/textures/water.jpg texture => taken from pintrest PublicDomainPictures.net: https://www.pinterest.com/pin/306737424617953515/
- res/textures/rockySurface.png texture => taken from pintrest Medialoot: https://www.pinterest.com/pin/22-beautiful-and-seamless-rock-textures-medialoot--570127634082430201/
- Heightmap textures taken from: https://tangrams.github.io/heightmapper/
