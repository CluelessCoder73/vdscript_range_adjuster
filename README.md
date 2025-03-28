vdscript_range_adjuster
This script is designed to adjust cut points in VirtualDub & VirtualDub2 script files (.vdscript) to ensure they align with legal frame boundaries, particularly useful when working with proxy videos for editing high-resolution footage. It guarantees that no frames are lost in the process, unlike most "stream copy" video editors. No need for aligning cut points with keyframes etc, because this script does all that for you automatically! After generating the adjusted .vdscript file, you can convert it to "Cuttermaran" or "LosslessCut" project files. I have created python scripts which can do exactly that, & they are available at https://github.com/CluelessCoder73?tab=repositories
This script now works in batch mode!
Tested and works with:
- Python 3.13.2
- VirtualDub 1.10.4 .vdscript files
- VirtualDub2 (build 44282) .vdscript files
- "FFmpeg" generated frame log files (the version in LosslessCut 3.64.0)

Features:

- Adjusts start points to previous I-frames. If the start point is already on an I-frame, it is left untouched. 
Alternatively, you can also adjust the start point to the "2nd" previous I-frame (the I-frame before the previous one). In that case, if the start point is already on an I-frame, it is instead adjusted to just the previous I-frame. Can be useful when working with x265 & other "open GOP" codecs, where cut-in points end up corrupted, & the video doesn't play right again until the next I-frame.
In fact, you can go furter back in I-frames, but I don't see any need (so far) to go any further back than 2.

- Adjusts endpoints to the next P or I-frame. If the endpoint is already on a P or I-frame, it is left untouched.
Alternatively, you can also adjust the endpoint to the last P-frame before the next I-frame ("short_cut_mode = False"). In that case, if, e.g., the endpoint is already on the last P-frame before the next I-frame, it is left untouched.
    
- Merges overlapping or close ranges (optional)

Prerequisites

Python 3.x installed on your system
Input .vdscript file(s) from VirtualDub or VirtualDub2 (sourcevideofilename.vdscript)
Frame log file (sourcevideofilename_frame_log.txt) containing frame type information

Configuration
At the bottom of the script, you'll find several configurable parameters:

directory = '.'
i_frame_offset = 1
merge_ranges_option = True
min_gap_between_ranges = 100
short_cut_mode = True

Adjust these parameters as needed:

directory: Defaults to current directory, change if needed
i_frame_offset: Number of I-frames to go back for start points (default: 1)
merge_ranges_option: Set to True to enable merging of close ranges, False to disable
min_gap_between_ranges: Minimum gap (in frames) to keep ranges separate when merging
short_cut_mode: Set to True to enable moving endpoints to the next P or I-frame, False for "full GOP mode"

Output
The script generates new .vdscript files with the adjusted cut points. These files can then be used directly in VirtualDub or VirtualDub2 (depending on which version created the original vdscript files!), or converted to other formats like .cpf (Cuttermaran project files) or .llc (LosslessCut project files).
Tips for Optimal Use

When editing proxy videos, place cut points freely without worrying about exact frame types.
Use this script to adjust the cut points before applying them to your high-resolution footage.
Experiment with the i_frame_offset value to find the best balance between accuracy and avoiding potential corruption from open GOP structures.
If you notice any issues with playback at cut points, try increasing the i_frame_offset value.

Troubleshooting

If the script fails to run, ensure you have Python 3.x installed.
If cut points seem incorrect, double-check your frame log to ensure it matches your video file.
For videos with unusual GOP structures, you may need to adjust the i_frame_offset.

Converting Output to Other Formats
After generating the adjusted .vdscript file, you can convert it to other formats:

For Cuttermaran: Use "vdscript_to_cpf" to create a .cpf file.
For LosslessCut: Use "vdscript_to_llc" to transform the .vdscript into a .llc file format.
Both are available at https://github.com/CluelessCoder73?tab=repositories

This script provides a powerful solution for ensuring accurate, lossless cuts in your video editing workflow, especially when working with proxy videos for high-resolution content. By automating the adjustment of cut points to legal frame boundaries, it saves time and guarantees the integrity of your final edit.


How to edit a 4K video using the proxy method:

Here's my guide on editing a 4K video in VirtualDub2, & saving the final export with LosslessCut. Because this method uses proxy videos, it does not require a high-end PC! NOTE: If your proxy videos are lagging in VirtualDub2, you will need to reduce the max resolution for the proxy presets!
Software/python scripts required:
HandBrake
VirtualDub2
LosslessCut
vdscript_info.py (optional)
vdscript_to_llc_v1.3.0.py

Step 1:
Put all your source videos into folders according to their frame rates (e.g., 23.976, 25 etc). For the sake of simplicity, for the rest of this guide, I will only refer to one folder, because the method for all folders is the same.

Step 2:
Create proxy versions of your videos using HandBrake: Use one of the provided custom presets. You may want to raise the "Constant Quality" values, because they are all set at "RF 16". The default "RF 22", or higher will be good enough for most. You may also want to lower the "Resolution Limit", which is set at "720p HD". NOTE: All filters are turned off, so if your video is e.g. interlaced, you will need to enable deinterlacing! DO NOT save to the same folder as your input files!

Step 3:
Open "frame_log_extractor.bat" in a text editor, & specify the path to "ffmpeg.exe". Hint: The version in LosslessCut should work just fine. You can now save the modified version of it for future use. Now copy the following scripts into your "source videos" folder:

1stGOP_analyzer_batch.py (optional)
frame_log_extractor.bat
vdscript_range_adjuster.py
vdscript_info.py (optional)
vdscript_to_llc_v1.3.0.py

Step 4:
Run "frame_log_extractor.bat". Be patient, it will take a long time. It will process every video it finds in the folder. Each frame log file will have the same name as its corresponding video (including extension), with "_frame_log.txt" appended.

Step 5:
Edit your proxy videos with VirtualDub2. You can use 32 or 64 bit, the output vdscript is identical. But for performance, I always use the 64 bit version. You will notice that these proxy versions are really easy to work with - you can scan at high speed through the videos by using SHIFT+LEFT & SHIFT+RIGHT, & you can go even faster by using ALT+LEFT & PGDOWN. Save your work in VirtualDub2 by using CTRL+S to save processing settings. MAKE SURE to check "Include selection and edit list", Otherwise your cuts will NOT be saved!!! Once you do that, it will remain so for future sessions. When editing is complete, the vdscript must be saved as "source video filename" + ".vdscript". So, if your source video is called "whatever.mp4", your final saved vdscript should be called "whatever.mp4.vdscript".

Step 6:
Run "vdscript_range_adjuster.py". It will process every vdscript it finds, & the outputted files will have "_adjusted.vdscript" appended.

Step 7:
Run "1stGOP_analyzer_batch.py" (optional - only for perfectionists). The "Smallest starting GOP" value indicates the minimum "minus" value you can enter in the "extra_frames_start" parameter in "vdscript_to_llc_v1.3.0.py" without risking the loss of the first GOP in any segment. Hint: It only processes files with "_adjusted.vdscript" appended, & it outputs a single file called "gop_info.txt".

Step 8: Run vdscript_info.py (optional) for a detailed "before & after" comparison. For "fps", enter the same as reported in LosslessCut for the ORIGINAL video - NOT the proxy! ("advanced view" is required for this). Do not use MediaInfo either, because it sometimes differs slightly from FFmpeg in its handling of frame rates.

Step 9:
Open "vdscript_to_llc_v1.3.0.py" in a text editor & edit the paths etc. For "fps", see "Step 8". Don't forget it's the "_adjusted" vdscript you're looking for! Then run it. & voila! - you now have a LosslessCut project file!



VERY IMPORTANT! - DO NOT get your original videos & proxy videos mixed up!!

VERIFYING THAT THE SOURCE & PROXY MATCH:
Open your "whatevervideo_frame_log.txt", & go to the 2nd last line (the last line of actual text); Somewhere in this line, it will say, e.g. "n:58357".
Now, open your proxy video in VirtualDub2, & hit the [End] key. This will bring you to the last frame of the video. The display at the bottom should say, e.g. "Frame 58358". That is the total number of frames in your proxy video, & SHOULD be +1 (in comparison to that last frame reported in the frame log), because in VirtualDub2, the last frame is always an "empty" frame.
DO NOT compare the proxy with the actual source video itself! The frame counts will often match, but NOT ALWAYS! The important thing (in terms of frame accuracy) is that your frame logs & proxy videos match.