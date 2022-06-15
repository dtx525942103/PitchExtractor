# PitchExtractor
This repo contains the training code for deep neural pitch extractor for Voice Conversion (VC) and TTS used in [StarGANv2-VC](https://github.com/yl4579/StarGANv2-VC) and [StyleTTS](https://github.com/yl4579/StyleTTS). 

## Pre-requisites
1. Python >= 3.7
2. Clone this repository:
```bash
git https://github.com/yl4579/PitchExtractor.git
cd AuxiliaryASR
```
3. Install python requirements: 
```bash
pip install SoundFile torchaudio torch pyyaml click matplotlib librosa pyworld
```
4. Prepare your own dataset and put the `train_list.txt` and `val_list.txt` in the `Data` folder (see Training section for more details).

## Training
```bash
python train.py --config_path ./Configs/config.yml
```
Please specify the training and validation data in `config.yml` file. The data list format needs to be `filename.wav|anything`, see [train_list.txt](https://github.com/yl4579/StarGANv2-VC/blob/main/Data/train_list.txt) as an example (a subset for LJSpeech). Note that you can put anything after the filename because the training labels are generated ad-hoc.

Checkpoints and Tensorboard logs will be saved at `log_dir`. To speed up training, you may want to make `batch_size` as large as your GPU RAM can take. 

### IMPORTANT: DATA FOLDER NEEDS WRITE PERMISSION
Since both `harvest` and `dio` are relatively slow, we do have to save the computed F0 ground truth for later use. In MelDataset.py, it will write the computed F0 curve `_f0.npy` for each `.wav` file. This requires write permission in your data folder. 

### F0 Computation Details
In MelDataset.py, the F0 curves are computated using [PyWorld](https://github.com/JeremyCCHsu/Python-Wrapper-for-World-Vocoder), one using `harvest` and another using `dio`. Both methods are acoustic-based and are unstable under certain conditions. `harvest` is faster but fails more than `dio`, so we first try `harvest`. When `harvest` fails (determined by number of frames with non-zero values), it will compute the ground truth F0 labels with `dio`. If `dio` fails, the computed F0 will have `NaN` and will be replaced with 0. This is supposed to occur only occasionally and should not affect training because these samples are treated as noises by the neural network and deep learning models are kwown to even benefit from slightly noisy datasets. However, if a lot of your samples has this problem (say > 5%), please remove them from the training set so that the model does not learn from the failed examples. 

### Data Augmentation
Data augmentation is not included in this code. For better voice conversion results, please add your own data augmentation in MelDataset.py with [audiomentations](https://github.com/iver56/audiomentations).

## References
- [keums/melodyExtraction_JDC](https://github.com/keums/melodyExtraction_JDC)
- [kan-bayashi/ParallelWaveGAN](https://github.com/kan-bayashi/ParallelWaveGAN)
