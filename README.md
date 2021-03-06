# AnsiLove.framework

This is a Cocoa framework I consider as tribute to the good old days™. It's capable of rendering ANSi / ASCII art and it also handles SAUCE records. 

# Tell me more

There are two classes responsible for all the magic: `ALAnsiGenerator` and `ALSauceMachine`. The former, `ALAnsiGenerator` creates Retina-ready PNG images from ANSi source files. What with one thing and another, images are read-only. So if you're looking for something that generates output in real textmode, maybe as a NSAttributedString instance, you're wrong. However, if you're seeking the most complete and accurate rendering of ANSi art sources available these days, you came to the right place. The latter, `ALSauceMachine` is reading SAUCE records and returns these values as Objective-C properties. 

# Version info

Current framework release: `5.0.2` - based on: [AnsiLove/C](https://github.com/ByteProject/AnsiLove-C) `2.1.0`

# Features

Rendering of all known ANSi / ASCII art file types:

- ANSi (.ANS)
- Binary (.BIN)
- Artworx (.ADF)
- iCE Draw (.IDF)
- Xbin (.XB) [details](http://www.acid.org/info/xbin/xbin.htm)
- PCBoard (.PCB)
- Tundra (.TND) [details](http://sourceforge.net/projects/tundradraw)
- ASCII (.ASC)
- Release info (.NFO)
- Description in zipfile (.DIZ)

Files with custom suffix default to the ANSi renderer (e.g. ICE or CIA).

AnsiLove.framework is capabable of processing: 

- SAUCE records
- DOS and Amiga fonts (embedded binary dump) 
- iCE colors

Still not enough?

- Output files are highly optimized 4-bit images.
- Optionally generate proper Retina @2x.PNG files.
- Use custom objects for adjusting output results.
- Built-in support for rendering Amiga ASCII.
- Everything's Mac App Store conform and sandboxing compliant.
- ARC / Automatic Reference Counting roject.

# Documentation

Let's talk about using the framework in your own projects. First of all, AnsiLove.framework is intended to run on `OS X`, it won't work for `iOS`. You have to download the sources and compile the framework. At least OS X Mountain Lion and Xcode 5.x are necessary for compiling `as is`. The project file contains two build targets, the framework itself and a test app `AnsiLoveGUI`, the latter is optional. Select `AnsiLoveGUI` from the Schemes dropdown in Xcode if you desire to compile that one too. The test app is a good example of implementing AnsiLove.framework, it does not contain much code and what you find there is well commented. So `AnsiLoveGUI` might be your first place to play with the framework after reading this documentation. Being an ARC framework, the test app is a pure ARC project as well. Makes sense, right?

## Implementing the framework in your own sources

I assume you know how to add a framework to your own projects, so we just skip that step now.

Go to the header of the class you want to use the framework with. Import the framework like this:

	#import <Ansilove/AnsiLove.h>

Create an instance of `ALAnsiGenerator`:

		 ALAnsiGenerator *ansiGen = [ALAnsiGenerator new];


To transform ANSi source files into a beautiful images, `ALAnsiGenerator` comes with just one method you should know:

	(void)renderAnsiFile:(NSString *)inputFile
			  outputFile:(NSString *)outputFile
			  		font:(NSString *)font
					bits:(NSString *)bits
			   iceColors:(BOOL      )iceColors
			   	 columns:(NSString *)columns
				  retina:(BOOL      )generateRetina;

Note that generally all objects except `inputFile` are optional. AnsiLove.framework will silently consume `nil` and empty string values and will rely on it's built-in defaults in both cases.

## (NSString *)inputFile

The only necessary object you need to pass to `ALAnsiGenerator`. Well, that's logic. If there is no input file, what should be the output? I see you get it. Here is an example for a proper `inputFile` string:

	/Users/Stefan/Desktop/MyAnsiArtwork.ans
	
I recommend treating this string case-sensitive.  As you can see, that string explicitly needs to contain the path and the file name. Keep in mind that any `NSURL` needs to be converted to a string before passing to ALAnsiGenerator. NSURL has a method called `absoluteString` that can be used for easy conversion.

	NSURL 	 *myURL;
	NSString *urlString = [myURL absoluteString];

Note that AnsiLove.framework will resolve any tilde in path for you.

## (NSString *)outputFile

Formatting of string `outputFile` is identical to string `inputFile`. If you don't set this object, the framework will use the same path / file name you passed as `inputFile` string, addding .PNG suffix automatically. However, if you plan to write your images into a different directory and / or under a different filename, go ahead and customize this object. Just keep in mind that for custom paths the suffix will be added automatically as well.

## (NSString *)font

AnsiLove.framework comes with two font families both originating from the golden age of ANSi artists. These font families are `PC` and `AMIGA`, the latter restricted to 8-bit only. Let's have a look at the values you can pass as `font` string.

`PC` fonts can be (all case-sensitive):

- `80x25` (code page 437)
- `80x50` (code page 437, 80x50 mode)
- `baltic` (code page 775)
- `cyrillic` (code page 855)
- `french-canadian` (code page 863)
- `greek` (code page 737)
- `greek-869` (code page 869)
- `hebrew` (code page 862)
- `icelandic` (Code page 861)
- `latin1` (code page 850)
- `latin2` (code page 852)
- `nordic` (code page 865)
- `portuguese` (Code page 860)
- `russian` (code page 866)
- `terminus` (modern font, code page 437)
- `turkish` (code page 857)

`AMIGA` fonts can be (all case-sensitive):

- `amiga` (alias to Topaz)
- `microknight` (Original MicroKnight version)
- `microknight+` (Modified MicroKnight version)
- `mosoul` (Original mO'sOul font)
- `pot-noodle` (Original P0T-NOoDLE font)
- `topaz` (Original Topaz Kickstart 2.x version)
- `topaz+` (Modified Topaz Kickstart 2.x+ version)
- `topaz500` (Original Topaz Kickstart 1.x version)
- `topaz500+` (Modified Topaz Kickstart 1.x version)

If you don't set a `font` object either passing `nil` or an empty string to ALAnsiGenerator, AnsiLove.framework will generate images using `80x25`, which is the default DOS font.

## (NSString *)bits

Bits can be (all case-sensitive): 

- `8` (8-bit)
- `9` (9-bit)
- `ced`
- `transparent`
- `workbench`

Setting the bits to `9` will render the 9th column of block characters, so the output will look like it is displayed in real textmode. 

Setting the bits to `ced` will cause the input file to be rendered in black on gray, and limit the output to 78 columns (only available for `.ans` files). Used together with an `AMIGA` font, the output will look like it is displayed on Amiga.

Setting the bits to `workbench` will cause the input file to be rendered using Amiga Workbench colors (only available for `.ans` files).

Settings the bits to `transparent` will produce output files with transparent background (only available for `.ans` files).

## (BOOL)iceColors

Setting `iceColors` to `YES` will enable iCE color codes. On the opposite `NO` means that that `iceColors` are disabled, which is the default value. When an ANSi source was created using iCE colors, it was done with a special mode where the blinking was disabled, and you had 16 background colors available. Basically, you had the same choice for background colors as for foreground colors, that's iCE colors. But now the important part: when the ANSi source does not make specific use of iCE colors, you should NOT enable them. The file could look pretty weird in normal mode. So in most cases it's fine to turn iCE colors off. 

## (NSString *)columns

`columns` is only relevant for ANSi source files with `.BIN` extension and even for those files optional. In most cases conversion will work fine if you don't set this flag, the default value is `160` then. So please pass `columns` only to `.BIN` files and only if you exactly know what you're doing. The sun could explode or even worse: A KITTEN MAY DIE SOMEWHERE.

## (BOOL)generateRetina

If you set this to `YES` the Framework creates two output images, the regular file and a properly named and sized @2x.PNG image to the same location.

## Supported options for each file type

Here's a simple overview of which file type supports which object. Note that ADF, IDF and XB sources don't support custom font objects as they come with embedded fonts:

	 ___________________________________________
	|     |         |       |       |           |
	|     | columns |  font |  bits | icecolors |
	|_____|_________|_______|_______|___________|
	|     |         |       |       |           |	
	| ANS |         |   X   |   X   |     X     |
	|_____|_________|_______|_______|___________|
	|     |         |       |       |           |	
	| PCB |         |   X   |   X   |     X     |
	|_____|_________|_______|_______|___________|
	|     |         |       |       |           |
	| BIN |    X    |   X   |   X   |     X     |
	|_____|_________|_______|_______|___________|
	|     |         |       |       |           |
	| ADF |         |       |       |           |
	|_____|_________|_______|_______|___________|
	|     |         |       |       |           |
	| IDF |         |       |       |           |
	|_____|_________|_______|_______|___________|
	|     |         |       |       |           |
	| TND |         |   X   |   X   |           |
	|_____|_________|_______|_______|___________|
	|     |         |       |       |           |
	| XB  |         |       |       |           |
	|_____|_________|_______|_______|___________|

## Rendering Process Feedback

AnsiLove.framework will post a notification once processing of given source files is finished. In most cases it's pretty important to know when rendering is done. One might want to present an informal dialog or update the UI afterwards. For making your app listen to the `AnsiLoveFinishedRendering` note, all you need is adding this to the `init` method of the class you consider as relevant:

    NSNotificationCenter *nc = [NSNotificationCenter defaultCenter];
    [nc addObserver:self 
           selector:@selector(addYourCustomSelectorHere:)
               name:@"AnsiLoveFinishedRendering"
             object:nil];

The test app `AnsiLoveGUI` has a simple implementation that posts a message to NSLog as soon as the rendering is completed. 

## Output file example

You may wonder how the rendered output looks like? You'll find an example in regular resolution [here](http://cl.ly/1D0o1M2t2Y190v33462F/o).

# Reading SAUCE records

The framework's class for dealing with SAUCE records is `ALSauceMachine`. But before we continue, here's your opportunity to introduce yourself to the [SAUCE specifications](http://www.acid.org/info/sauce/s_spec.htm). Plenty values retrieved from SAUCE records can be passed as objects to `ALAnsiGenerator`, so it makes sense indeed to check for a SAUCE record before you start rendering. So, how to use the class? First, we need to create an instance of `ALSauceMachine`:

	 ALSauceMachine *sauce = [ALSauceMachine new];

Of course `[[ALSauceMachine alloc] init];` will work fine as well. Now call `readRecordFromFile:`, this should be self-explanatory:

	[sauce readRecordFromFile:myInputFile];

You probably guess that not all files contain SAUCE and you're right. Many ANSi files actually contain SAUCE (like [this file](http://sixteencolors.net/pack/acid-56/W7-R666.ANS)) and some just don't. So how do you know? I've implemented three handy BOOL values, they give you all the feedback you need: 

	BOOL fileHasRecord;
	BOOL fileHasComments;
	BOOL fileHasFlags;

The first property, `fileHasRecord` is kinda superior, which means if a file doesn't have a SAUCE record, it's evident it doesn't have SAUCE comments and flags. My advice: don't check `fileHasComments` and `fileHasFlags` if you already know there is no SAUCE record. What if a file contains a SAUCE record on the other hand? It's nevertheless possible it doesn't have comments and flags. So if `fileHasRecord` is YES, you should try the other two BOOL values as well. Let's assume your class instance is still `*sauce` and you now checked for all three BOOL types, knowing the details. How to retrieve the SAUCE? Easy. `ALSauceMachine` stores the SAUCE record into properties, right at your fingertips:

	NSString  *ID;
	NSString  *version;
	NSString  *title;
	NSString  *author;
	NSString  *group;
	NSString  *date;
	NSInteger dataType;
	NSInteger fileType;
	NSInteger tinfo1;
	NSInteger tinfo2;
	NSInteger tinfo3;
	NSInteger tinfo4;
	NSString  *comments;
	NSInteger flags;

Now imagine you want to print SAUCE title and author in NSLog:

	NSLog(@"This is: %@ by %@.", sauce.title, sauce.author);

That's it. If you feel like this introduction to AnsiLove.framework's SAUCE implementation left some of your questions unanswered, I suggest you take a closer look at `AnsiLoveGUI` (the sample app). It contains a full featured yet simple example how to use `ALSauceMachine`. 

# App Sandboxing

The framework runs great in sandboxed apps. That is because I handcrafted it to be like that. No temporary exceptions, no hocus-pocus, just you on your lonely island. AnsiLove.framework comes with it's own tiny subsystem to achieve sandboxing compliance. Sounds like no big deal? Go sit on a tack. Actually that was the hardest part of the whole framework.

# Retina support

You already know this framework comes with full Retina support. Assuming you are familiar with Apple's [High Resolution Guidelines for OS X](http://developer.apple.com/library/mac/#documentation/GraphicsAnimation/Conceptual/HighResolutionOSX/Introduction/Introduction.html), there is not much more to say about the matter. Now it's up to you as developer.

# Why?

AnsiLove.framework was created for my app [Escapes](http://escapes.byteproject.net).

# Credits

I'd like to thank my friends [Frederic Cambus](http://www.cambus.net) and [Brian Cassidy](http://blog.alternation.net/) for their ongoing support. Both had a major impact on [AnsiLove/C](https://github.com/ByteProject/AnsiLove-C) and thus on AnsiLove.framework. While Fred is also responsible for [AnsiLove/C's](https://github.com/ByteProject/AnsiLove-C) well-known ancestor [AnsiLove/PHP](http://ansilove.sourceforge.net/), significant parts of Brian's [libsauce](https://github.com/bricas/libsauce) breathe life into `ALSauceMachine`. Finally I bow to all the great ANSi / ASCII lovers and artists around the world. You are the artscene. You keep alive what was not meant to die years ago. YOU ARE ROCKSTARS!

# License

Ascension is released under a MIT-style license. See the file `LICENSE` for details.
