![MusicLang logo](https://github.com/MusicLang/musiclang/blob/main/documentation/images/MusicLang.png?raw=true "MusicLang")

<h1 align="center" weight='300' >MusicLang Predict, your controllable music copilot. </h1>

<h4 align="center">
  <a href="https://huggingface.co/musiclang/musiclang-v2"> 🤗 HuggingFace</a> |
  <a href="https://discord.gg/2g7eA5vP">Discord</a> |
  <a href="https://www.linkedin.com/company/musiclang/">Follow us!</a>
</h4>
<br/>

<h4>☞ You want to generate music that you can export to your favourite DAW in MIDI ?</h4> 
<h4>☞ You want to control the chord progression of the generated music ?</h4> 
<h4>☞ You need to run it fast on your laptop without a gpu ?</h4> 

<br/>
<h2 align="center"><b>MusicLang is the contraction of “Music” & “language”: we bring advanced controllability features over music generation by manipulating symbolic music.</b></h2>
<br/>

<summary><kbd>Table of contents</kbd></summary>

- <a href="#quickstart">Quickstart 🚀</a>
    - <a href="#colab">Try it quickly 📙</a>
    - <a href="#install">Install MusicLang ♫</a>
- <a href="#examples">Examples 🎹</a>
    - <a href="#example_1">1. Generate your first music 🕺</a>
    - <a href="#example_2">2. Controlling chord progression generation 🪩</a>
    - <a href="#example_3">3. Generation from an existing music 💃</a>
- <a href="#next">What's coming next at musiclang? 👀</a>
- <a href="#hdw">How does MusicLang work? 🔬</a>
    - <a href="#article_1">1. Annotate chords and scales progression of MIDIs using MusicLang analysis</a>
    - <a href="#article_2">2. The MusicLang tokenizer : Toward controllable symbolic music generation</a>
    - <a href="#article_3">3. Examples of sound made with MusicLang ❤️</a>
- <a href="#share">Contributing & spread the word 🤝</a>
- <a href="#license">License ⚖️</a>

<h1 id="quickstart">Quickstart 🚀</h1>
<h2 id="colab">Try it quickly 📙</h2>
<br/>

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1MA2mek826c05BjbWk2nRkVv2rW7kIU_S?usp=sharing)
[![Open in Spaces](https://huggingface.co/datasets/huggingface/badges/resolve/main/open-in-hf-spaces-sm.svg)](https://huggingface.co/spaces/musiclang/musiclang-predict)

Go to our <a href="https://colab.research.google.com/drive/1MA2mek826c05BjbWk2nRkVv2rW7kIU_S?usp=sharing">♾Colab</a>, or to our <a href="https://huggingface.co/spaces/musiclang/musiclang-predict">🤗HuggingFace space</a>, we have a lot of cool examples, from generating creative musical ideas to continuing a song with a specified chord progression.
<br/>

<h2 id="install">Install MusicLang ♫</h2> 
<br/>

Install the `musiclang-predict` package :

```bash
pip install musiclang_predict
pip install scipy==1.12.0
```


- Render In:     
- https://huggingface.co/spaces/asigalov61/Advanced-MIDI-Renderer

<h2 id="examples">Examples 🎹</h2>

<h3 id="example_1">1. Generate your first music 🕺</h3> 
<br/>

Open your favourite notebook and start generating music in a few lines :

```python
from musiclang_predict import MusicLangPredictor
nb_tokens = 1024 
temperature = 0.9  # Don't go over 1.0, at your own risks !
top_p = 1.0 # <=1.0, Usually 1 best to get not too much repetitive music
seed = 16  # change here to change result, or set to 0 to unset seed

ml = MusicLangPredictor('musiclang/musiclang-v2') # Only available model for now

score = ml.predict(
    nb_tokens=nb_tokens,  # 1024 tokens ~ 25s of music (depending of the number of instruments generated)
    temperature=temperature,
    topp=top_p,
    rng_seed=seed # change here to change result, or set to 0 to unset seed
)
score.to_midi('test.mid') # Open that file in your favourite DAW, score editor or even in VLC
```

<h3 id="example_2">2. Controlling chord progression generation 🪩</h3>
<br/>

You had a specific harmony in mind, right ? MusicLang allows fine control over the chord progression of the generated music.
Just specify it as a string like below, choose a time signature and let the magic happen.

```python
from musiclang_predict import MusicLangPredictor

# Control the chord progression
# Chord qualities available : M, m, 7, m7b5, sus2, sus4, m7, M7, dim, dim0.
# You can also specify the bass if it belongs to the chord (eg : Bm/D)
chord_progression = "Am CM Dm E7 Am" # 1 chord = 1 bar
time_signature = (4, 4) # 4/4 time signature, don't be too crazy here 
nb_tokens = 1024 
temperature = 0.8
top_p = 1.0
seed = 42

ml = MusicLangPredictor('musiclang/musiclang-v2')

score = ml.predict_chords(
    chord_progression,
    time_signature=time_signature,
    temperature=temperature,
    topp=top_p,
    rng_seed=seed # set to 0 to unset seed
)
score.to_midi('test.mid', tempo=120, time_signature=(4, 4))
```

> Disclaimer : The chord progression is not guaranteed to be exactly the same as the one you specified. It's a generative model after all. This may occur more frequently when using an exotic chord progression or setting a high temperature.

<h3 id="example_3">3. Generation from an existing music 💃</h3>
<br/>

What if I want to use MusicLang from an existing music ? Don't worry, we got you covered. You can use your music as a template to generate new music.
Let's continue with some Bach music and explore a chord progression he might have used: 
```python
from musiclang_predict import MusicLangPredictor
from musiclang_predict import corpus

song_name = 'bach_847' # corpus.list_corpus() to get the list of available songs
chord_progression = "Cm C7/E Fm F#dim G7 Cm"
nb_tokens = 1024 
temperature = 0.8 
top_p = 1.0 
seed = 3666 

ml = MusicLangPredictor('musiclang/musiclang-v2')

score = ml.predict_chords(
    chord_progression,
    score=corpus.get_midi_path_from_corpus(song_name),
    time_signature=(4, 4),
    nb_tokens=1024,
    prompt_chord_range=(0,4),
    temperature=temperature,
    topp=top_p,
    rng_seed=seed # set to 0 to unset seed
)

score.to_midi('test_bash.mid', tempo=110, time_signature=(4, 4))
```

- 可用的不同音乐风格
```python
from musiclang.analyze.chord_repr_to_chord import chord_repr_list_to_chords

chord_progressions = {
    "流行经典": ["CM", "GM", "Am", "FM"],
    "爵士风情": ["Dm7", "G7", "CM7", "FM7"],
    "阳光旋律": ["EM", "GM", "AM", "CM"],
    "布鲁斯律动": ["A7", "D7", "A7", "E7", "D7", "A7", "E7"],
    "温暖叙事": ["GM", "DM", "Em", "CM"],
    "忧郁温暖": ["Am", "GM", "FM", "EM"],
    "灵魂深处": ["Fm", "AbM", "EbM", "DbM"],
    "乡村风情": ["GM", "CM", "DM", "GM"],
    "爵士夜晚": ["Am7", "D7", "GM7", "CM7"],
    "情感流淌": ["BbM7", "Gm7", "Cm7", "F7"]
}

for k, v in chord_progressions.items():
    print(k)
    print(chord_repr_list_to_chords(v))

from musiclang_predict import MusicLangPredictor, corpus
from tqdm import tqdm
import os

# 定义和弦进行字典
chord_progressions = {
    "流行经典": ["CM", "GM", "Am", "FM"],
    "爵士风情": ["Dm7", "G7", "CM7", "FM7"],
    "阳光旋律": ["EM", "GM", "AM", "CM"],
    "布鲁斯律动": ["A7", "D7", "A7", "E7", "D7", "A7", "E7"],
    "温暖叙事": ["GM", "DM", "Em", "CM"],
    "忧郁温暖": ["Am", "GM", "FM", "EM"],
    "灵魂深处": ["Fm", "AbM", "EbM", "DbM"],
    "乡村风情": ["GM", "CM", "DM", "GM"],
    "爵士夜晚": ["Am7", "D7", "GM7", "CM7"],
    "情感流淌": ["BbM7", "Gm7", "Cm7", "F7"]
}

# 初始化 MusicLangPredictor
ml = MusicLangPredictor('musiclang/musiclang-v2')

# 设置参数
song_name = 'bach_847'  # 从 corpus.list_corpus() 获取可用歌曲列表
nb_tokens = 1024
temperature = 0.8
top_p = 1.0
seed = 3666
output_dir = 'output_midi_files'  # 指定保存 MIDI 文件的路径

# 创建输出目录
os.makedirs(output_dir, exist_ok=True)

# 使用 tqdm 迭代和弦进行字典
for name, progression in tqdm(chord_progressions.items(), desc="生成 MIDI 文件"):
    # 将和弦列表转换为字符串
    chord_progression_str = " ".join(progression)
    
    # 生成音乐
    score = ml.predict_chords(
        chord_progression_str,
        score=corpus.get_midi_path_from_corpus(song_name),
        time_signature=(4, 4),
        nb_tokens=nb_tokens,
        prompt_chord_range=(0, 4),
        temperature=temperature,
        topp=top_p,
        rng_seed=seed  # 设置为 0 以取消种子
    )
    
    # 保存 MIDI 文件
    output_path = os.path.join(output_dir, f"{name}.mid")
    score.to_midi(output_path, tempo=110, time_signature=(4, 4))

print(f"所有 MIDI 文件已保存到目录: {output_dir}")

!zip -r output_midi_files.zip output_midi_files
```

- Genshin Impact （Venti/Kaveh） Music demo
```
git clone https://huggingface.co/datasets/svjack/dialogue_feat_merge_save_unique
cp dialogue_feat_merge_save_unique/提瓦特音乐（人物）（新）.zip .
unzip -O GBK 提瓦特音乐（人物）（新）.zip
cp 提瓦特音乐（人物）（新）/温迪.mp3 .

pip install basic-pitch
mkdir output_midi
basic-pitch output_midi 温迪.mp3

find "提瓦特音乐（人物）（新）" -type f -name "*.mp3" -exec basic-pitch output_midi {} \;
```

- MIDI Render APP
```bash
git clone https://huggingface.co/spaces/svjack/Advanced-MIDI-Renderer && cd Advanced-MIDI-Renderer && pip install -r requirements.txt
python demo_app.py
```
```python
from gradio_client import Client, handle_file

client = Client("http://127.0.0.1:7860")
mid_file_name = "output_midi/卡维_basic_pitch.mid"
result = client.predict(
		input_midi=handle_file(mid_file_name),
		render_type="Render as-is",
		soundfont_bank="Super GM",
		render_sample_rate="16000",
		custom_render_patch=-1,
		render_align="Do not align",
		render_transpose_value=0,
		render_transpose_to_C4=False,
		render_output_as_solo_piano=False,
		render_remove_drums=False,
		api_name="/Render_MIDI"
)
#print(result)
from shutil import copy2
copy2(result[4], "render.wav")
from IPython import display
display.Audio("render.wav")

!ffmpeg -i render.wav -codec:a libmp3lame -qscale:a 2 卡维_midi_render.mp4 -y
!ffmpeg -i "提瓦特音乐（人物）（新）/卡维.mp3" -codec:a libmp3lame -qscale:a 2 卡维.mp4 -y
```



https://github.com/user-attachments/assets/608bdea1-98f6-47b1-a82c-9413545ce826



https://github.com/user-attachments/assets/6892f1fb-f2f5-41bc-b9fe-cb3b91e1f489

- Kaveh Transform
```python
from musiclang_predict import MusicLangPredictor, corpus
from tqdm import tqdm
import os

# 定义和弦进行字典
chord_progressions = {
    "流行经典": ["CM", "GM", "Am", "FM"],
    "爵士风情": ["Dm7", "G7", "CM7", "FM7"],
    "阳光旋律": ["EM", "GM", "AM", "CM"],
    "布鲁斯律动": ["A7", "D7", "A7", "E7", "D7", "A7", "E7"],
    "温暖叙事": ["GM", "DM", "Em", "CM"],
    "忧郁温暖": ["Am", "GM", "FM", "EM"],
    "灵魂深处": ["Fm", "AbM", "EbM", "DbM"],
    "乡村风情": ["GM", "CM", "DM", "GM"],
    "爵士夜晚": ["Am7", "D7", "GM7", "CM7"],
    "情感流淌": ["BbM7", "Gm7", "Cm7", "F7"]
}

# 初始化 MusicLangPredictor
ml = MusicLangPredictor('musiclang/musiclang-v2')

# 设置参数
#song_name = 'bach_847'  # 从 corpus.list_corpus() 获取可用歌曲列表
nb_tokens = 1024
temperature = 0.8
top_p = 1.0
seed = 3666
output_dir = 'kaveh_midi_files'  # 指定保存 MIDI 文件的路径

# 创建输出目录
os.makedirs(output_dir, exist_ok=True)

# 使用 tqdm 迭代和弦进行字典
for name, progression in tqdm(chord_progressions.items(), desc="生成 MIDI 文件"):
    # 将和弦列表转换为字符串
    chord_progression_str = " ".join(progression)
    
    # 生成音乐
    score = ml.predict_chords(
        chord_progression_str,
        score="output_midi/卡维_basic_pitch.mid",
        time_signature=(4, 4),
        nb_tokens=nb_tokens,
        prompt_chord_range=(0, 4),
        temperature=temperature,
        topp=top_p,
        rng_seed=seed  # 设置为 0 以取消种子
    )
    
    # 保存 MIDI 文件
    output_path = os.path.join(output_dir, f"{name}.mid")
    score.to_midi(output_path, tempo=110, time_signature=(4, 4))

print(f"所有 MIDI 文件已保存到目录: {output_dir}")

!zip -r kaveh_midi_files.zip kaveh_midi_files
```

- Kaveh Transform MP4
```python
import os
from gradio_client import Client, handle_file
from shutil import copy2
from tqdm import tqdm  # 导入 tqdm

# 初始化 Gradio 客户端
client = Client("http://127.0.0.1:7860")

# 定义输入和输出路径
midi_dir = "kaveh_midi_files"  # MIDI 文件所在的目录
output_dir = "kaveh_output_mp4"  # 输出 MP4 文件的目录

# 确保输出目录存在
os.makedirs(output_dir, exist_ok=True)

# 获取所有 MIDI 文件
midi_files = [f for f in os.listdir(midi_dir) if f.endswith(".mid")]

# 使用 tqdm 显示进度条
for midi_file in tqdm(midi_files, desc="Processing MIDI files"):
    mid_file_path = os.path.join(midi_dir, midi_file)
    
    # 使用 Gradio 客户端渲染 MIDI 文件
    result = client.predict(
        input_midi=handle_file(mid_file_path),
        render_type="Render as-is",
        soundfont_bank="Super GM",
        render_sample_rate="16000",
        custom_render_patch=-1,
        render_align="Do not align",
        render_transpose_value=0,
        render_transpose_to_C4=False,
        render_output_as_solo_piano=False,
        render_remove_drums=False,
        api_name="/Render_MIDI"
    )
    
    # 将渲染后的 WAV 文件复制到当前目录
    wav_output_path = "render.wav"
    copy2(result[4], wav_output_path)
    
    # 定义输出 MP4 文件的路径
    output_mp4_path = os.path.join(output_dir, f"卡维_{midi_file.replace('.mid', '.mp4')}")
    
    # 使用 ffmpeg 将 WAV 文件转换为 MP4 文件
    os.system(f'ffmpeg -i {wav_output_path} -codec:a libmp3lame -qscale:a 2 {output_mp4_path} -y')
    
    # 清理临时文件（可选）
    os.remove(wav_output_path)

print("All files processed! Check the output_mp4 directory.")

!zip -r kaveh_output_mp4.zip kaveh_output_mp4
```


<h2 id="next">What's coming next at MusicLang? 👀</h2>
<br/>

We are working on a lot of cool features, some are already encoded in the model :
- A control over the instruments used in each bar and their properties (note density, pitch range, average velocity);
- Some performances improvements over the inference C script;
- A faster distilled model for real-time generation that can be embedded in plugins or mobile applications;
- An integration into a DAW as a plugin;
- Some specialized smaller models depending on our user's needs;
- And more to come! 😎

<h2 id="hdw">How does MusicLang work? 🔬</h2>
<br/>

If you want to learn more about how we are moving toward symbolic music generation, go to our [technical blog](https://musiclang.github.io/). The tokenization, the model are described in great details: 

<h4 id="article_1"><a href="https://musiclang.github.io/chord_parsing/"> 1. Annotate chords and scales progression of MIDIs using MusicLang analysis</a></h4>
<h4 id="article_2"><a href="https://musiclang.github.io/tokenizer/"> 2. The MusicLang tokenizer : Toward controllable symbolic music generation</a></h4>
<h4 id="article_3"><a href="https://musiclang.github.io/generations/"> 3. Examples of sound made with MusicLang ❤️</a></h4>
<br/>

We are using a LLAMA2 architecture (many thanks to Andrej Karpathy's awesome [llama2.c](https://github.com/karpathy/llama2.c)), trained on a large dataset of midi files (The CC0 licensed [LAKH](https://colinraffel.com/projects/lmd/)).
We heavily rely on preprocessing the midi files to get an enriched tokenization that describe chords & scale for each bar.
The is also helpful for normalizing melodies relative to the current chord/scale.


<h2 id="share">Contributing & spread the word 🤝</h2> 
<br/>

We are looking for contributors to help us improve the model, the tokenization, the performances and the documentation.
If you are interested in this project, open an issue, a pull request, or even [contact us directly](https://www.musiclang.io/contact).

Whether you're contributing code or just saying hello, we'd love to hear about the work you are creating with MusicLang. Here's how you can reach out to us: 
* Join our [Discord](https://discord.gg/2g7eA5vP) to ask your questions and get support
* Follow us on [Linkedin](https://www.linkedin.com/company/musiclang/)
* Add your star on [GitHub](https://github.com/musiclang/musiclang_predict?tab=readme-ov-file) or [HuggingFace](https://huggingface.co/musiclang/musiclang-4k)

<h2 id="license">License ⚖️</h2>
<br/>

MusicLang Predict (This package) is licensed under the GPL-3.0 License.
However please note that specific licenses applies to our models. If you would like to use the model in your commercial product, please
[contact us](https://www.musiclang.io/contact). We are looking forward to hearing from you !

The MusicLang base language package on which the model rely ([musiclang package](https://github.com/musiclang/musiclang)) is licensed under the BSD 3-Clause License.
