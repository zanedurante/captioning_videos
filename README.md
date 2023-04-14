# captioning_videos
Quick code for captioning videos

This snippet uses code from MoviePy but removes the ImageMagick dependency!

## Installation

`pip install moviepy numpy pillow`

## Code snippet

```
from moviepy.editor import VideoFileClip, ImageClip, CompositeVideoClip
from PIL import Image, ImageDraw, ImageFont
import numpy as np

# load the video file
test_caps = [
    {"text": "Caption 1 testing the captioning capabilities", "start": 0, "end": 4},
    {"text": "Caption 2 now that is all done with, let's test the REAL captioning!", "start": 5, "end": 8},
]
def caption_video(video_path="examples/clip_09.mp4", captions=test_caps):
    output_vid = video_path[video_path.rfind("/"):]
    video = VideoFileClip(video_path)

    # create a blank image with the same dimensions as the video
    w, h = video.size

    # create a text clip for each caption
    text_clips = []
    mask_clips = [video]
    for caption in captions:
        txt = Image.new("RGBA", (w, h), (0, 0, 0, 0))
        draw = ImageDraw.Draw(txt)
        # draw the text onto the blank image
        font = ImageFont.truetype("/usr/share/fonts/truetype/dejavu/DejaVuSerif.ttf", size=50)
        draw.text((w/3, h-80), caption["text"], font=font, fill=(255, 255, 255))

        # create a mask image from the text image
        mask = Image.fromarray(np.uint8(np.array(txt)[:,:,:] > 0) * 255)

        # create an ImageClip from the mask image
        mask_clip = ImageClip(np.array(mask))
        mask_clip = mask_clip.set_position(('center', 'bottom')).set_duration(caption["end"] - caption["start"])
        mask_clip = mask_clip.set_start(caption["start"])
        mask_clips.append(mask_clip)

        # create a CompositeVideoClip with the original video and the mask clip
        #text_clips.append(CompositeVideoClip([video, mask_clip]))
    final_clip = CompositeVideoClip(mask_clips)
    # combine the video and text clips
    #final_clip = CompositeVideoClip(text_clips)

    # write the final video to a file
    final_clip.write_videofile("outputs/{}".format(output_vid))
```
