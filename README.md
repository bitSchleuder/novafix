# novafix

A quick and dirty fix for EV Nova on OS X El Capitan.

## The Problem

	2015-10-07 22:52:07.530 EV Nova[17753:652894] 22:52:07.530 WARNING:  140: This application, or a library it uses, is using the deprecated Carbon Component Manager for hosting Audio Units. Support for this will be removed in a future release. Also, this makes the host incompatible with version 3 audio units. Please transition to the API's in AudioComponent.h.
	2015-10-07 22:52:32.576 EV Nova[17753:652894] Error loading /Applications/EV Nova.app/Contents/Frameworks/ASWCarbonSparkleBridge.bundle/Contents/MacOS/ASWCarbonSparkleBridge:  dlopen(/Applications/EV Nova.app/Contents/Frameworks/ASWCarbonSparkleBridge.bundle/Contents/MacOS/ASWCarbonSparkleBridge, 262): Symbol not found: _NewOTProcessUPP
	  Referenced from: /Applications/EV Nova.app/Contents/Frameworks/ASWCarbonSparkleBridge.bundle/Contents/MacOS/ASWCarbonSparkleBridge
	  Expected in: /System/Library/Frameworks/CoreServices.framework/Versions/A/CoreServices
	dyld: lazy symbol binding failed: Symbol not found: _CGSSetWindowDepthLimit
	  Referenced from: /Applications/EV Nova.app/Contents/MacOS/EV Nova
	  Expected in: /System/Library/Frameworks/ApplicationServices.framework/Versions/A/ApplicationServices
	
	dyld: Symbol not found: _CGSSetWindowDepthLimit
	  Referenced from: /Applications/EV Nova.app/Contents/MacOS/EV Nova
	  Expected in: /System/Library/Frameworks/ApplicationServices.framework/Versions/A/ApplicationServices
	
	Trace/BPT trap

Particularly the last bit about `CGSSetWindowDepthLimit`. That's what keeps it from running, since it appears this function (which seems to not be public API) was silently removed from El Capitan.

## The Solution

The Console message gives you the problem. By disassembling `CGSSetWindowDepthLimit` on a machine that still has it, you can see it's a no-op, so injecting one doesn't hurt.

libNova.A.dylib is a dylib containing just a function named `CGSSetWindowDepthLimit`. Using dyld's environment variables, it is injected before anything else, and the namespace is flattened, via launcher.sh.

If your copy of EV Nova happens to not be in `/Applications/EV Nova.app`, simply adjust the path at the end of launcher.sh.
	
## A note to Ambrosia SW

This is easy to fix. Simply remove the call to CGSSetWindowDepthLimit. It is already a no-op anyways.

	CoreGraphics`CGSSetWindowDepthLimit:
	CoreGraphics[0x3bed8a]:  pushq  %rbp
	CoreGraphics[0x3bed8b]:  movq   %rsp, %rbp
	CoreGraphics[0x3bed8e]:  xorl   %eax, %eax
	CoreGraphics[0x3bed90]:  popq   %rbp
	CoreGraphics[0x3bed91]:  retq   

P.S. I don't actually expect to fix it. I've been watching your site, I know EV is abandonware, that's why this exists.