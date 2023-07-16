# MP4 (Dolby Vision)

## Create MP4 container (Dolby Vision Profile 8.1)

### (1) Dolby Vision Profile 8.1 (AC-3)
.\dovi_tool --mode 2 --drop-hdr10plus mux --discard --bl 00000.track_4113.hevc --el 00000.track_4117.hevc --output Movie.HDR.DV81.hevc

.\dovi_tool demux Movie.hevc
.\dovi_tool --mode 2 --drop-hdr10plus mux --discard --bl BL.hevc --el EL.hevc --output Movie.HDR.DV81.hevc

.\ffmpeg-audio-normalizer `
    -i Movie.ac3 `
    -o Movie.Dub.DN-31.ac3 `
    dialogue --target-level -31

.\ffmpeg-audio-normalizer `
    -i Movie.ac3 `
    -o Movie.Dub.DN-31.EBUR128.ac3 `
    ebu `
    -- -dialnorm -31

.\mp4muxer2 `
    --dvp 8 --dv-bl-compatible-id 1 --mpeg4-comp-brand mp42,iso6,isom,msdh,dby1 `
    -i Movie.hevc -n "mp4muxer2 v2.2.5 (based on https://github.com/DolbyLaboratories/dlb_mp4base)" `
    -i Movie.Dub.DN-31.ac3 -n "Dub, Blu-ray (DN -31dB)" -l rus `
    -i Movie.Dub.DN-31.EBUR128.ac3 -n "Dub, Blu-ray (DN -31dB, EBU R 128)" -l rus `
    -o Movie.2160p.HDR.DV81.Dub.DN-31.EBUR128.mp4 --overwrite `
&& `
.\MP4Box.exe `
    Movie.2160p.HDR.DV81.Dub.DN-31.EBUR128.mp4 `
    -add Movie.srt:lang=rus:name="Forced"

.\MP4Box.exe `
    Movie.2160p.HDR.DV81.Dub.DN-31.EBUR128.mp4 `
    -new -brand mp42isom -ab dby1 `
    -add Movie.hevc:dvp=f8.hdr10:xps_inband:hdr=none `
    -add Movie.Dub.DN-31.ac3:lang=rus:name="Dub, Blu-ray (DN -31dB)" `
    -add Movie.Dub.DN-31.EBUR128.ac3:lang=rus:name="Dub, Blu-ray (DN -31dB, EBU R 128)" `
    -add Movie.srt:lang=rus:name="Forced"


### (2) Prepare Audio

#### Dialogue normalization
.\ffmpeg-audio-normalizer `
    -i Movie.ac3 `
    -o Movie.DN-31.ac3 `
    dialogue --target-level -31

#### Convert any audio -> E-AC-3 5.1 with dialog normalization (-31dB)
.\ffmpeg-audio-normalizer `
    -i Movie.dts `
    -o Movie.DN-31.eac3 `
    dialogue --target-level -31 `
    -- "-c:a" eac3 "-b:a" 1509k

#### Convert any audio -> E-AC-3 5.1 with normalization EBU R128 and dialog normalization (-31dB)
.\ffmpeg-audio-normalizer `
    --verbose `
    -i Movie.dts `
    -o Movie.DN-31.EBUR128.eac3 `
    ebu `
    -- "-c:a" eac3 "-b:a" 1509k -dialnorm -31

ffmpeg-normalize `
    Movie.dts `
    -o Movie.DN-31.EBUR128.eac3 `
    --verbose --progress --print-stats `
    --normalization-type ebu `
    -c:a eac3 -b:a 1509k -ar 48000 -e="-dialnorm -31"

#### Dialog normalization (-31dB) for AC-3 5.1
.\ffmpeg.exe `
    -i Movie.ac3 `
    -c:a ac3 -b:a 640k -dialnorm -31 `
    Movie.DN-31.ac3

#### Normalize AC-3 5.1 to EBU R128 with dialog normalization (-31dB)
.\ffmpeg-audio-normalizer `
    -i Movie.ac3 `
    -o Movie.DN-31.EBUR128.ac3 `
    ebu -- `
    -dialnorm -31

ffmpeg-normalize `
    Movie.ac3 `
    -o Movie.DN-31.EBUR128.ac3 `
    --verbose --progress --print-stats `
    --normalization-type ebu `
    -c:a ac3 -b:a 640k -ar 48000 -e="-dialnorm -31"

### (3) Prepare video
":dvp=8.hdr10" -> hvc1
":dvp=8.hdr10:xps_inband" -> hev1
*":dvp=8.hdr10:xps_inband:hdr=none" -> hev1
":dvp=8.hdr10 --xps_inband=both" -> hev1
":dvp=f8.hdr10" -> dvh1
":dvp=f8.hdr10:xps_inband" -> dvhe
":dvp=f8.hdr10 --xps_inband=both" -> dvhe
*":dvp=f8.hdr10:xps_inband:hdr=none" -> dvhe

#### Convert Blu-ray BL HEVC, EL+RPU HEVC -> BL+RPU HEVC
.\dovi_tool --mode 2 extract-rpu --input 00000.track_4117.hevc --rpu-out RPU.bin `
&& `
.\dovi_tool --mode 2 --drop-hdr10plus inject-rpu --input 00000.track_4113.hevc --rpu-in RPU.bin --output BL_RPU.hevc


### (3.1) GPAC MP4Box mux MP4 container (Dolby Vision Profile 8.1) with E-AC-3 audio
MP4Box.exe `
    Movie.HDR.DV81.DN-31.EBUR128.mp4 `
    -new -brand mp42isom -ab dby1 `
    -add Movie.hevc:dvp=f8.hdr10:xps_inband:hdr=none `
    -add Movie.eac3:lang=rus:name="Dub, Blu-ray":sopt:gfreg=ffdmx `
    -add Movie.DN-31.EBUR128.eac3:lang=rus:name="Dub, Blu-ray (DN -31dB, EBU R128)":sopt:gfreg=ffdmx `
    -add Movie.srt:lang=rus:name="Forced"

### (3.2) MP4Muxer mux MP4 container (Dolby Vision Profile 8.1) with E-AC-3 audio
.\mp4muxer.exe `
    --locale "ru-RU" `
    --dvp 8 --dv-bl-compatible-id 1 --mpeg4-comp-brand mp42,iso6,isom,msdh,dby1 `
    --input-file Movie.hevc --media-name "Muxed by MP4muxer v1.2.1 (Dolby Laboratories, Inc.)" `
    --input-file Movie.DN-31.eac3 --media-lang rus --media-name "Dub, Blu-ray (DN -31dB)" `
    --input-file Movie.DN-31.EBUR128.eac3 --media-lang rus --media-name "Dub, Blu-ray (DN -31dB, EBU R128)" `
    --output-file Movie.HDR.DV81.DN-31.EBUR128.mp4 --overwrite `
&& `
MP4Box.exe `
    Movie.HDR.DV81.DN-31.EBUR128.mp4 `
    -add Movie.srt:lang=rus:name="Forced"

### (3.3) GPAC MP4Box mux MP4 container (Dolby Vision Profile 8.1) with AC-3 audio
MP4Box.exe `
    Movie.HDR.DV81.DN-31.EBUR128.mp4 `
    -new -brand mp42isom -ab dby1 `
    -add Movie.hevc:dvp=f8.hdr10:xps_inband:hdr=none `
    -add Movie.ac3:lang=rus:name="Dub, Blu-ray" `
    -add Movie.DN-31.EBUR128.ac3:lang=rus:name="Dub, Blu-ray (DN -31dB, EBU R128)" `
    -add Movie.srt:lang=rus:name="Forced"

### (3.4) MP4Muxer mux MP4 container (Dolby Vision Profile 8.1) with AC-3 audio
.\mp4muxer.exe `
    --locale "ru-RU" `
    --dvp 8 --dv-bl-compatible-id 1 --mpeg4-comp-brand mp42,iso6,isom,msdh,dby1 `
    --input-file Movie.hevc --media-name "Muxed by MP4muxer v1.2.1 (Dolby Laboratories, Inc.)" `
    --input-file Movie.ac3 --media-lang rus --media-name "Dub, Blu-ray" `
    --input-file Movie.DN-31.ac3 --media-lang rus --media-name "Dub, Blu-ray (DN -31dB)" `
    --input-file Movie.DN-31.EBUR128.ac3 --media-lang rus --media-name "Dub, Blu-ray (DN -31dB, EBU R128)" `
    --output-file Movie.HDR.DV81.DN-31.EBUR128.mp4 --overwrite `
&& `
MP4Box.exe `
    Movie.HDR.DV81.DN-31.EBUR128.mp4 `
    -add Movie.srt:lang=rus:name="Forced"


-------------------------------------------------------------------------------


## Others

### Audio

#### Extract DTS Core from DTS-HD Master Audio
.\ffmpeg.exe `
    -i Movie.dts `
    -c:a copy -bsf:a dca_core `
    Movie.DTSCore.dts

#### Convert any 5.1 audio -> E-AC-3 5.1 with normalization RMS -23dB and dialog normalization (-31dB)
.\ffmpeg-audio-normalizer `
    --verbose `
    -i Movie.dts `
    -o Movie.RMS-23.eac3 `
    rms --target-level -23 `
    -- "-c:a" eac3 -dialnorm -31

ffmpeg-normalize `
    Movie.dts `
    -o Movie.RMS-23.eac3 `
    --verbose --progress --print-stats `
    --normalization-type rms --target-level -23 `
    -c:a eac3 -b:a 1509k -ar 48000 -e="-dialnorm -31"

#### Convert any 5.1 audio -> E-AC-3 5.1 with normalization Peak 0dB and dialog normalization (-31dB)
.\ffmpeg-audio-normalizer `
    --verbose `
    -i Movie.dts `
    -o Movie.Peak0.eac3 `
    peak --target-level 0 `
    -- "-c:a" eac3 -dialnorm -31

ffmpeg-normalize `
    Movie.dts `
    -o Movie.Peak0.eac3 `
    --verbose --progress --print-stats `
    --normalization-type peak --target-level 0 `
    -c:a eac3 -b:a 1509k -ar 48000 -e="-dialnorm -31"

#### Convert any 5.1 audio -> AC-3 5.1 with dialog normalization (-31dB)
.\ffmpeg.exe `
    -i Movie.dts `
    -c:a ac3 -b:a 640k -dialnorm -31 `
    Movie.ac3

#### Normalize AC-3 5.1 to RMS -23dB with dialog normalization (-31dB)
.\ffmpeg-audio-normalizer `
    --verbose `
    -i Movie.ac3 `
    -o Movie.RMS-23.ac3 `
    rms --target-level -23 `
    -- -dialnorm -31

ffmpeg-normalize `
    Movie.ac3 `
    -o Movie.RMS-23.ac3 `
    --verbose --progress --print-stats `
    --normalization-type rms --target-level -23 `
    -c:a ac3 -b:a 640k -ar 48000 -e="-dialnorm -31"

#### Normalize AC-3 5.1 to Peak 0dB with dialog normalization (-31dB)
.\ffmpeg-audio-normalizer `
    --verbose `
    -i Movie.ac3 `
    -o Movie.Peak0.ac3 `
    peak --target-level -23 `
    -- -dialnorm -31

ffmpeg-normalize `
    Movie.ac3 `
    -o Movie.Peak0.ac3 `
    --verbose --progress --print-stats `
    --normalization-type peak --target-level 0 `
    -c:a ac3 -b:a 640k -ar 48000 -e="-dialnorm -31"

#### Add delay to AC-3
.\ffmpeg.exe `
    -itsoffset 500ms `
    -i Movie.ac3 `
    -c:a ac3 -b:a 640k -dialnorm -31 `
    Movie.DN-31.ac3


### Video

#### Demux BL+EL+RPU HEVC -> BL HEVC, EL+RPU HEVC
.\dovi_tool demux Movie.hevc

#### Changing the frame rate
.\ffmpeg -i Movie.hevc -filter:v fps=30 Movie.30fps.hevc

### MP4 Container

#### Add AC3 audios (original, RMS -23dB, Peak 0dB, EBU R128) and SRT subtitles to MP4
.\MP4Box.exe `
    Movie.RMS-23.Peak0.EBUR128.mp4 `
    -add Movie.ac3:lang=rus:name="Dub, Blu-ray" `
    -add Movie.RMS-23.ac3:lang=rus:name="Dub, Blu-ray (RMS -23dB)" `
    -add Movie.Peak0.ac3:lang=rus:name="Dub, Blu-ray (Peak 0dB)" `
    -add Movie.EBUR128.ac3:lang=rus:name="Dub, Blu-ray (EBU R128)" `
    -add Movie.RMS-23.Peak0.EBUR128.srt:lang=rus:name="Forced"

#### Mux MP4 container (Dolby Vision Profile 7)
.\mp4muxer.exe `
    --locale "ru-RU" `
    --dvp 7 --mpeg4-comp-brand mp42,iso6,isom,msdh,dby1 `
    --input-file Movie.hevc --media-name "Muxed by MP4muxer v1.2.1 (Dolby Laboratories, Inc.)" `
    --input-file Movie.eac3 --media-lang rus --media-name "Dub, Blu-ray" `
    --input-file Movie.EBUR128.eac3 --media-lang rus --media-name "Dub, Blu-ray (EBU R128)" `
    --output-file Movie.HDR.DV81.EBUR128.mp4 --overwrite `
&& `
.\MP4Box.exe `
    Movie.HDR.DV81.EBUR128.mp4 `
    -add Movie.srt:lang=rus:name="Forced"

#### Mux MP4 container (Dolby Vision Profile 5)
.\mp4muxer.exe `
    --locale "ru-RU" `
    --dvp 5 --mpeg4-comp-brand mp42,iso6,isom,msdh,dby1 `
    --input-file Movie.hevc --media-name "Muxed by MP4muxer v1.2.1 (Dolby Laboratories, Inc.)" `
    --input-file Movie.eac3 --media-lang rus --media-name "Dub, Blu-ray" `
    --input-file Movie.EBUR128.eac3 --media-lang rus --media-name "Dub, Blu-ray (EBU R128)" `
    --output-file Movie.HDR.DV81.EBUR128.mp4 --overwrite `
&& `
.\mp4box.exe `
    Movie.HDR.DV81.EBUR128.mp4 `
    -add Movie.srt:lang=rus:name="Forced"

#### Demux MP4 (DL demuxer)
.\mp4demuxer.exe `
    --input-file Movie.mp4 `
    --output-folder .\

#### Demux MP4 (ffmpeg)
.\ffmpeg `
    -i Movie.mp4 `
    -map 0:0 -c copy Movie.hevc `
    -map 0:1 -c copy Movie.ac3

#### Extract SRT from MP4
.\mp4box.exe `
    Movie.mp4 `
    -srt 10
