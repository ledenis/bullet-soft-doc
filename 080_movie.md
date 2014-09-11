Generating a movie
==================

Creating a movie file of a simulation can be done in different ways. One can capture using an external program or directly from the simulation program itself. The latter is really practical but requires this feature to be implemented. Unfortunately, Bullet does not provide a solution to do it. 

The approach chosen here consists in reading the OpenGL frames (as they are rendered on screen), storing them as *PNG* image files in the hard drive, then generating the movie file from the images. *PNG* format is better than other ones such as *JPEG*, *GIF*, *BMP* due to its losless compression, which means the image quality is preserved through the size compression.

Preparation
-----------

### Tools used

To save the *PNG* files, the official **libpng** is chosen for this task. It is the most natural solution. Libpng depends on zlib, a compression library. The version used in this chapter is 1.6.13, with zlib 1.2.8.

The movie encoding task is accomplished by the tool **ffmpeg** as a command line executable. It can be directly downloaded on its [official website][ffmpeg-download] under the *Get the packages* section. On Windows, it is built daily.

Libpng needs to be compiled, the source can be found on [sourceforge.net][libpng-sourceforge]. Zlib binaries can be downloaded on the [official website][zlib-website].

Compiling libpng with CMake

* Download compiled zlib
* Put the folder in root of libpng (next to cmakelists)
* Rename zlib folder to "zlib"
* In CMakeLists add at beginning:
set(ZLIB_ROOT zlib)
* Compile (cmake then make)


Usage:
include:
<libpng source>/
<libpng build>/ (pnglibconf.h is generated here)

lib:
<libpng build>/
png

copy libpng16.dll

-------------------------------

Implementation
--------------

### Reading the OpenGL frames

The first step is to read the pixels of the OpenGL frame. This is done with the function `glReadPixels()`. Its prototype is:

	void glReadPixels(GLint x, GLint y, GLsizei width, GLsizei height,
			GLenum format, GLenum type, GLvoid * data);

It needs an array as a buffer to store all the pixels. For example, we store it as an array of *unsigned byte* (`GLubyte`), in the *RGB* format. The entire frame is needed, hence the parameters *(0, 0, width, height)*.

	// get screen pixels into buffer
	GLubyte* buffer = new GLubyte[width * height * 3]; // 3 for R, G and B
	glReadPixels(0, 0, width, height, GL_RGB, GL_UNSIGNED_BYTE, buffer);


### Storing as PNG images

Then, we store it by using the *libpng* API. The simplified API, which is released with the 1.6.0 version, is used here. As its name suggests, the simplified API simplifies considerably the PNG read/write workflow compared to the legacy API. It relies mostly on the single structure `png_image`. According to the [libpng manual][libpng-manual]:

>To write a PNG file using the simplified API:
>
>  1) Declare a 'png_image' structure on the stack and memset()
>     it to all zero.
>
>  2) Initialize the members of the structure that describe the
>     image, setting the 'format' member to the format of the
>     image in memory.
>
>  3) Call the appropriate png_image_write... function with a
>     pointer to the image to write the PNG data.

The steps 1 and 2 can be implemented as follows:

	// write to png file
	png_image image;
	memset(&image, 0, sizeof image);
	image.version = PNG_IMAGE_VERSION;
	image.format = PNG_FORMAT_RGB;
	image.width = width;
	image.height = height;

The function `png_image_write_to_file()` is used at the third step, its prototype is:

	int png_image_write_to_file (png_imagep image,
	      const char *file, int convert_to_8bit, const void *buffer,
	      png_int_32 row_stride, const void *colormap);

We do not use the two last parameters, thus we set them to 0.

	char filename[50];
	sprintf(filename, "screen%s_%d.png", m_timeStr, m_frameNb);
	m_frameNb++;
	png_image_write_to_file(&image, filename, 0, buffer, 0, 0);

	delete buffer;

Calling all these functions each iteration of a simulation results as a series of images created in the disk space. To reconstruct a movie from these files, it is important to assign numbers sequentially to them, hence the inclusion of a counter `m_frameNb` into the file name. Though it is optional, a timestep `m_timeStr` is used to prevent unwanted behaviour if images of previous movie recording remain.



-------------------------



				// get screen pixels into buffer
				GLubyte* buffer = new GLubyte[width * height * 4]; // 3 for R, G and B
				glReadPixels(0, 0, width, height, GL_RGBA, GL_UNSIGNED_BYTE, buffer);

				// write to png file
				png_image image;
				memset(&image, 0, sizeof image);
				image.version = PNG_IMAGE_VERSION;
				image.format = PNG_FORMAT_RGBA;
				image.width = width;
				image.height = height;

				char* filename = new char[16];
				static int frameNb = 0;
				sprintf(filename, "screen%d.png", frameNb);
				frameNb++;
				cout << filename << endl;
				png_image_write_to_file(&image, filename, 0, // convert to 8 bit
						buffer, 0, // row stride
						0); // color map
				cout << image.message << endl;
				cout << "test" << endl;

				delete buffer;

----------------				

### Generating the movie file

To generate the movie file, ffmpeg is one 
ffmpeg

----------------------

-f image2
Demuxer from image files
To use when converting images files into video

-r 25
Framerate: (before -i for input, before output file name for output)

-i 'screen_%03d.jpg'
Input file name's pattern: %03d means 0-padded number with 3 digits, like 001, 002, ..., 999

-start_number 1
Start number for the pattern

-vf 'vflip'
Flip vertically

-y
Overwrite if output file exists

video.avi
Output file name

	ffmpeg -f image2 -r 1/2 -i 'screen_%03d.jpg' -start_number 1 -y -r 5 video.avi

-----------------------

[ffmpeg-download]: http://ffmpeg.org/download.html
[libpng-sourceforge]: http://sourceforge.net/projects/libpng/files/
[zlib-website]: http://www.zlib.net/
[libpng-manual]: http://www.libpng.org/pub/png/libpng-manual.txt
