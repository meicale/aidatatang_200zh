# Sunflower Collection of Kaldi-based aidatatang_200zh Training

[Translation](https://github.com/datatang-ailab/aidatatang_200zh/blob/master/README.zh.md)


## Description

- Test environment: Ubuntu 16.04

- Instruction: this is a sample of speech recognition benchmark experiment released as per aidatatang_200zh, Datatang open source Chinese Mandarin Corpus

- Update time of document: May 10, 2019

- Speech recognition accuracy: 94.41%

## Introduction of Kaldi

Kaldi is a free and open source speech recognition toolkit written by C++, with Apache v2.0 open source protocol. The Kaldi toolkit features modern and flexible code that is easy to be modified and extended. It is internally integrated with Finite State Transducers (FSTs), and is provided with a wide range of linear algebra support and complete sample script of speech recognition, which creates a good condition for the development and learning of speech recognition technology.

Please visit [here](http://kaldi-asr.org/) for a brief introduction of Kaldi

Please visit [here](http://kaldi-asr.org/doc/)for official document of Kaldi

Please visit [here](<https://github.com/kaldi-asr/kaldi>) for Kaldi code and version control

## Installation of Kaldi

**Installation Procedures**

Download relevant dependent packages before formal installation:

```shell
sudo apt-get install autoconf automaker gcc g++ libtool subversion gawk
sudo apt-get install libatlas-dev libatlas-base-dev gfortran zlib1g-dev
```

Download the latest Kaldi toolkit, go to the Kaldi installation path, and install the Kaldi toolkit as per instructions in the INSTALL file under the directory.

```shell
git clone https://github.com/kaldi-asr/kaldi.git kaldi --origin upstream
cd kaldi
```

First, go to the tools directory and carry out corresponding steps as per instructions of tools/INSTALL.

Before that, dependencies must be checked:

```shell
cd tools
./extras/check_dependencies.sh
```

If there is no problem with the check result, move to the compilation step. First, enter the following command to check the number of CPU cores in the computer:

```shell
nproc
```

Suppose the output is 4, the compilation parameter can be set to 4 to speed up the compilation:

```shell
make -j 4
```

Note that IRSTLM is no longer installed by default in the current Kaldi version. If required, you can install IRSTLM by yourself, as shown below:

```shell
./extras/install_irstlm.sh 
```

Next, enter the SRC directory and enter the following command separately in the command lines as per instructions of src/INSTALL:

```shell
 ./configure --shared
  make depend -j 4
  make -j 4
```

This is the compilation stage. It takes a long time to compile the downloaded package into an executable file. Please be patient.

Above instructions only apply to UNIX-like systems. Please refer to the *Windows/INSTALL* file for the environment compilation of Windows system.

**Explanations of files in Kaldi**

- /egs: Experimental sample scripts base on various corpora

- /tools: Dependency library used for Kaldi storage and installation

- /src: used to store source code and executable files

**Kaldi installation verification**

In order to verify whether the Kaldi toolkit is successfully installed, run the egs/yesno/example and the following command:

```shell
cd egs/yesno/s5
./run.sh
```

The smooth obtaining of identification results means that the Kaldi toolkit has been successfully installed and you can carry out the formal speech recognition research based on Kaldi toolkit! Congratulations!

## aidatatang_200zh script training

### General overview

Enter the egs/aidatatang_200zh/ directory, and you can see README file and s5 folder.

```shell
cd egs/aidatatang_200zh/
```

README document summarizes the content and download method of the aidatatang_200 corpus. The s5 directory contains all the configuration files and scripts related to the corpus-based speech recognition experiments. 

There are several folders and files under s5 directory, and their brief introductions are as follows:

| Folder    | Description                        | Remark                                              |
| --------- | ---------------------------------- | --------------------------------------------------- |
| **conf**  | Configuration directory            | Feature configuration document                      |
| **local** | Script directory                   | Scripts required for specific projects              |
| **steps** | Script directory                   | Data processing tool provided by Kaldi              |
| **utils** | Script directory                   | Model tool provided by Kaldi                        |
| cmd.sh    | Hardware configuration             | Single cluster configuration                        |
| path.sh   | Environment variable configuration | Import environment variable                         |
| run.sh    | General scripts                    | General project running script                      |
| RESULTS   | Experimental result (CER)          | List the recognition results of all training models |

i.e.,: in s5/conf are the configuration documents used for model training.

​       While in s5/{local, steps, utils} are the scripts used for run.sh.

Before formal running of script, first confirm whether you want to run the experiment on a single machine/server or in a cluster. If the former, modify the configuration of cmd.sh, as shown below:

```shell
cd s5
vim cmd.sh
### Change queue.pl into run.pl
```

Next, open up the general project script and modify the data set save path, i.e.:

```shell
vim run.sh
### Replace data=/export/corpora/aidatatang with the path that you want to save the data set to
```

Finally, run the script and you will get the speech recognition effect based on aidatatang_200zh corpus as shown in the RESULTS:

```shell
./run.sh
```

Note: it is recommended to run the run.sh script one line after another, so as to clearly understand the process at each stage.

### Detailed explanation of training

Theoretically, run the run.sh script and you can complete the entire training steps as long as relevant environment is configured correctly. However, in order to understand the training process of speech recognition better, the understanding of every detail and step in the training process is essential. In the sample script include four stages: data download, data preparation, GMM-HMM model training and DNN-HMM model training. The following text will introduce the detail tasks performed in each phase one line after another.

**Environment Configuration**

The speech recognition system training cannot be carried out until necessary environment variables are activated first, which are separately the Kaldi running environment variable declared in path. sh and the single cluster configuration stated in cmd. sh, as shown below:

```shell
. ./cmd.sh ## You'll want to change cmd.sh to something that will work on your system.
           ## This relates to the queue.
. ./path.sh
```

**Download and decompression of data set**

Then, complete the data preprocessing step at the beginning of the speech recognition system, which will download the Datatang open source Chinese Mandarin Corpus aidatatang_200zh data set from the specified address and save in the specified directory.

```shell
# corpus directory and download URL
data=/export/corpora/aidatatang/
data_url=www.openslr.org/resources/62

# You can obtain the database by uncommting the follwing lines
[ -d $data ] || mkdir -p $data || exit 1;
local/download_and_untar.sh $data $data_url data_aidatatang || exit 1;
```

After successful download of data set, the data preparation phase begins.

**Data preparation**

 Step 1: to prepare relevant mapping files for running the Kaldi program, including:

- text: containing transcripts for each piece of speech, with the format of ＜utterance-id＞＜transcription＞.

- wav.scp: index file recording the speech location, with the format of ＜utterance-id＞＜filename＞.

- utt2spk: to indicate the corresponding relationship between speech and speaker, with the format of ＜utterance-id＞＜speaker-id＞.

- spk2utt: to indicate the corresponding relationship between speaker and speech, which can be generated by utt2spk, with the format of ＜speaker-id＞＜utterance-id＞.

```shell
# Data Preparation: generate text, wav.scp, utt2spk, spk2utt
local/aidatatang_data_prep.sh $data/corpus $data/transcript || exit 1;
```

New folder will be generated under the current directory after the execution of this command, with relevant mapping files checked through looking over the files under the directory of data/{local}{}/{train,dev,test}.

**Dictionary Preparation**

Step 2: to produce a dictionary containing all transcripts in the corpus.

CMU English dictionary and CEDIT Chinese-English dictionary are downloaded and merged here to generate a Chinese-English dictionary.

```shell
# Lexicon Preparation: build a large lexicon that invovles words in both the training and decoding
local/aidatatang_prepare_dict.sh || exit 1;
```

The process document generating the dictionary is saved in the folder of data/local/dict, while the resulting dictionary in the data/local/dict/lexicon.txt.

**Preparation of language model**

Step 3: to prepare a language model for speech recognition training according to the transcripts in the corpus. The process is divided into the following two stages:

- Prepare the decision tree questions set of Triphone Model and compile L.fst, in which, L.fst is used to map the phoneme sequences to word sequences.

```shell
# Prepare Language Stuff
# Phone Sets, questions, L compilation
utils/prepare_lang.sh --position-dependent-phones false data/local/dict "<UNK>" data/local/lang data/lang || exit 1;
```

- Trigram language model is trained to generate G.fst, and L.fst and G.fst are spliced to generate LG.fst. Among them, LG.fst can find the most likely word sequence corresponding to a given phoneme sequence according to Trigram model.

```shell
# LM training
local/aidatatang_train_lms2.sh || exit 1;
# G compilation, check LG composition
local/aidatatang_format_data.sh
```

The process files generated in this phase are saved in the directories data/lang and data/lang_test, and relevant models can be further examined.

Thus, the data preparation has been completed, and next the GMM-HMM model training based on aidatatang_200zh corpus will continue.

**Extraction of Features**

Before the training of GMM-HMM model, we first need to extract features from the original speech data. What we extract here is the MFCC plus pitch feature.

```shell
# Now make MFCC plus pitch features.
# mfccdir should be some place with a largish disk where you
# want to store MFCC features.
mfccdir=mfcc
for x in train dev test; do
  steps/make_mfcc_pitch.sh --cmd "$train_cmd" --nj 10 data/$x exp/make_mfcc/$x $mfccdir || exit 1;
  steps/compute_cmvn_stats.sh data/$x exp/make_mfcc/$x $mfccdir || exit 1;
  utils/fix_data_dir.sh data/$x || exit 1;
done
```

The feature files generated are saved under the directory mfcc, while the log files of the calculation process are saved in the directory exp/make_mfcc.

Enter the directory mfcc, you can see multiple .ark files containing features and .scp files containing feature indexes. The binary feature file is the default type generated by the program, which can be converted into plain text format by the following instructions:

```shell
copy-feats ark:raw_mfcc_pitch_dev.1.ark ark,t:raw_mfcc_pitch_dev.1.txt
```

**GMM-HMM Model Training**

The training is divided into five stages, including the training of monophone hidden Markov model (train_mono.sh), training of context-sensitive triphone model (train_deltas.sh), training of triphone model on linear discriminant analysis and maximum likelihood linear transformation (train_lda_mllt.sh), training of triphone model on speaker self-adaption (train_sat.sh) and maximum likelihood linear regression alignment based on feature space (align_fmllr.sh).

The following text explains the training process in detail by taking the training of monophone hidden Markov model as an example.

- The acoustic model is trained as per training data and language model:

```shell
steps/train_mono.sh --cmd "$train_cmd" --nj 10 
data/train data/lang exp/mono || exit 1;
```

The final model of training is exp/mono/final.mdl, in which the parameters of GMM model are saved. The contents of the model can be viewed by using the following command:

```shell
gmm-copy --binary=false exp/mono/final.mdl - | less
```

- Construct the monophone decoding graph:

```shell
# Monophone decoding
utils/mkgraph.sh data/lang_test exp/mono exp/mono/graph || exit 1;
```

The program mainly generates HCLG.fst and words.txt files, which play a key role in the subsequent training stage.

- Decoding: specific to development set and test set respectively

```shell
# Monophone decoding
steps/decode.sh --cmd "$decode_cmd" --config conf/decode.config --nj 10 
  exp/mono/graph data/dev exp/mono/decode_dev
steps/decode.sh --cmd "$decode_cmd" --config conf/decode.config --nj 10 
  exp/mono/graph data/test exp/mono/decode_test
```

The decoding results and logs are saved under the directories of exp/mono/decode_dev and exp/mono/decode_test.

- Veterbi aligning

```shell
# Get alignments from monophone system.
steps/align_si.sh --cmd "$train_cmd" --nj 10 
  data/train data/lang exp/mono exp/mono_ali || exit 1;
```

In this stage, the training data is realigned by using the model already generated as the input to the new model, and the alignment results are saved in the directory exp/mono_ali.

The monophone training process is completed so far, with other training stages of the GMM-HMM model continued in a similar process, to generate the acoustic model and decoding results, saved in the exp directory.

**DNN-HMM Model Training**

In the whole training process, the output of DNN acoustic model can be obtained by GMM-HMM model, that is, DNN training depends on GMM-HMM model, so to train a good GMM-HMM model is one of the key factors to improve the effect of speech recognition. On the other hand, with the development of deep learning, DNN model presents the performance that obviously surpasses the GMM model and replaces GMM to realize HMM state modeling.

In this example, the TDNN time-delay neural network model and Chain model in Kaldi nnet3 are used to train and decode the data set, which is shown as below:

```shell
# nnet3
local/nnet3/run_tdnn.sh
```

```shell
# chain
local/chain/run_tdnn.sh
```

**Review Results**

```shell
# getting results (see RESULTS file)
for x in exp/*/decode_test; do [ -d $x ] && grep WER $x/cer_* | utils/best_wer.sh; done 2>/dev/null
```

### Summary 

By running the above GMM-HMM and DNN-HMM hybrid models, the experiment of speech recognition benchmark based on Datatang open source Chinese Mandarin Corpus aidatatang_200zh has been completed perfectly, and its character recognition accuracy is as follows:

<table>
   <tr>
      <td colspan=6 align="center">GMM-HMM(%)</td>
      <td colspan=2 align="center">DNN-HMM(%)</td>
   </tr>
   <tr>
      <td>mono</td>
      <td>Tri1</td>
      <td>Tri2</td>
      <td>Tri3a</td>
      <td>Tri4a</td>
      <td>Tri5a</td>
      <td>TDNN</td>
      <td>Chain</td>
   </tr>  
   <tr>
      <td>62.91</td>
      <td>82.02</td>
      <td>82.06</td>
      <td>82.74</td>
      <td>85.84</td>
      <td>87.78</td>
      <td>92.86</td>
      <td>94.41</td>
   </tr>
</table>

Where,

- mono refers to the result obtained from the training of monophone hidden Markov model;
-  Tri1 refers to the result obtained from the training of context-sensitive triphone model with mono model as input;
- Tri2 refers to the result obtained from the training of context-sensitive triphone model with Tri1 model as input;
- Tri3a refers to the result obtained from the training of triphone model on linear discriminant analysis and maximum likelihood linear transformation;
- Tri4a refers to the result obtained from the training of speaker self-adaption;
- Tri5a refers to the result obtained from deeper training of speaker self-adaption;
- TDNN refers to the result obtained from the training of TDNN-HMM;
- Chain refers to the result obtained from the training of Chain model;

According to the recognition effect of the benchmark experiment, the speech recognition accuracy based on aidatatang_200zh corpus can reach 87.78% after several rounds of GMM-HMM model training, which can reach up to 94.41% after DNN-HMM model training on this basis. It also indicates that good data support can be obtained for Chinese speech recognition research from the Chinese Mandarin corpus released by Datatang which contains 600 speakers from different regions of China, with a total length of 200 hours and a total of 237,265 voices after carefully annotated manually.

Although the development of speech technology has certain history, and the deep learning achievements in the field of speech recognition research has greatly promoted the development of speech technology, the data is still one of the limitation factors for voice technology. The open source of this data set has enriched the domestic mandarin corpora, and provided quality data resources for the majority of voice technology enthusiasts. Interested researchers are expected to download the aidatatang_200zh Chinese mandarin corpus for use.

## More Resource

Datatang (Beijing) Technology Co., Ltd is a professional AI data service provider and is dedicated to providing data collection, processing and data product services for global artificial intelligence enterprises, covering data types such as speech, image and text, including biometrics, speech recognition, autonomous driving, smart home, smart manufacturing, new retail, OCR scene, smart medical treatment, smart transportation, smart security, mobile phone entertainment, etc.

[For more open source data sets](https://www.datatang.com/webfront/opensource.html)

[For more business data sets](https://www.datatang.com/webfront/datatang_dataset.html)
