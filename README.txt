{\rtf1\ansi\ansicpg1252\cocoartf2868
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\froman\fcharset0 Times-Roman;\f1\fmodern\fcharset0 Courier;}
{\colortbl;\red255\green255\blue255;\red0\green0\blue0;\red109\green109\blue109;}
{\*\expandedcolortbl;;\cssrgb\c0\c0\c0;\cssrgb\c50196\c50196\c50196;}
\paperw11900\paperh16840\margl1440\margr1440\vieww11520\viewh8400\viewkind0
\deftab720
\pard\pardeftab720\sa240\partightenfactor0

\f0\fs24 \cf0 \expnd0\expndtw0\kerning0
\outl0\strokewidth0 \strokec2 Here's the README text to copy:\
\pard\pardeftab720\partightenfactor0
\cf3 \strokec3 \
\pard\pardeftab720\partightenfactor0

\f1\fs26 \cf0 \strokec2 MacBook Pro Keyboard Simulator\
==============================\
\
An interactive MacBook Pro keyboard built as a single HTML file.\
Press keys onscreen or use your physical keyboard to trigger animations,\
ripple waves, and musical notes.\
\
Features\
--------\
- Ripple wave animation spreading from any pressed key\
- Musical note playback via Tone.js synthesis (C major, 4 octaves)\
- Dark and light key colour modes\
- Holographic shimmer wave effect\
- 6 background gradients with smooth crossfade transitions\
- On-load keyboard reveal animation\
- Fully scalable \'97 adapts to any viewport width\
\
Controls\
--------\
Hover below the keyboard to reveal the controls panel:\
\
  Wave Speed    \'97 how fast the ripple travels across keys\
  Wave Reach    \'97 maximum distance the wave spreads\
  Key Min Size  \'97 minimum key scale during a wave\
  Key Max Size  \'97 maximum key scale at the wave origin\
  Glow Colour   \'97 colour of the flash overlay on each wave\
  Glow Opacity  \'97 intensity of the glow effect\
  Shimmer       \'97 toggles holographic shimmer on waves\
  Key Colour    \'97 Dark or Light key theme\
  Keyboard      \'97 show or hide the aluminium plate\
  Background    \'97 choose from 6 gradient backgrounds\
\
Running\
-------\
No build step or server required. Open keyboard.html directly in any\
modern browser, or serve locally:\
\
  python3 -m http.server 8790\
  # visit http://localhost:8790/keyboard.html\
\
Dependencies\
------------\
Loaded via CDN (internet connection required):\
  - Tone.js 15.0.4  \'97 audio synthesis\
  - Inter           \'97 UI font (Google Fonts)\
\
License\
-------\
GNU General Public License v3.0\
See LICENSE.txt for details.}