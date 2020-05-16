
# DNN-based ASR with UAspeech
- Baseline cfg file for UAspeech data using pytorch-kaldi based DNN's
- This is just an example on how to use the pytorch-kaldi library to improve the WER of dysarthric speech ASR.
- To use this toolkit you must already have alignments which can be generated from a GMM-HMM acoustic model with kaldi, check the following repositories if you have not already done so.
  - https://github.com/ffxiong/uaspeech
  - https://github.com/abnerLing/dysarthria-asr
  
  
### Installation
- install pytorch https://pytorch.org/get-started/locally/
- clone pytorch-kaldi repository (I highly reccomend following one of their tutorials on TIMIT or Librispeech first, so you know the basics of this toolkit).
```
git clone git clone https://github.com/mravanelli/pytorch-kaldi
```
install required libraries for pytorch-kaldi
```
cd pytorch-kaldi
pip install -r requirements.txt
```

### Instructions
1. The cfg I made uses fbank+pitch+fmllr features so you will need to extract those for all train/test/dev sets<br/>
To extract and concatenate fbank+pitch features you can run something like this:
```
steps/make_fbank_pitch.sh --nj $nj --cmd "$train_cmd" data/train exp/make_fbank_pitch/train $feadir
steps/compute_cmvn_stats.sh data/train exp/make_fbank_pitch/train $feadir
```
For extracting fmllr features you can run something like this (where gmmdir is the directory of your alignments and data_fmllr is where your fmllr features will be located):
```
steps/nnet/make_fmllr_feats.sh --nj $nj --cmd "$train_cmd" --transform-dir ${gmmdir}_ali \
        $data_fmllr/train $data_dir/train $gmmdir $data_fmllr/train/log $data_fmllr/train/data || exit 1
```
2. Next you need fmllr align train/test/dev sets
```
steps/align_fmllr.sh --nj $nj --cmd "$train_cmd" --boost-silence $boost_sil \
      $data_dir/train $lang ${gmmdir} exp/tri4_ali_train || exit 1
```
3. Directly modify the cfg directories to point to your features, alignments, scoring script<br/>
This will need to be done for all features, all alignments, all data sets.
```
features:
fea_lst=/home/user/kaldi/egs/uaspeech/data/train/fbank_pitch/feats.scp
alignments:
lab_folder=/home/user/kaldi/egs/uaspeech/exp/tri4_ali
scoring:
scoring_script = /home/user/kaldi/egs/uaspeech/local/score.sh
```
4. Run cfg script 
  - I recommend just working in the pytorch-kaldi directory.
``` 
python /pytorch-kaldi/run_exp.py cfg/ua_best.cfg
```
Check Results
``` 
cat /pytorch-kaldi/exp/uaspeech_best/decode_test_out_dnn1/scoring_kaldi/best_wer
%WER 32.74 [ 7686 / 23477, 0 ins, 0 del, 7686 sub ]
```

## Modify hyperparameters?
There are many things you could test out to improve WER as I did not have time to optimize all hyperparameters.
- Adjust # of layers or hidden nodes.
- Modify learning rate or the scheduler.
- Different DNN model types (I found the RNN-based models did not improve results same as in [2]).
- Different optimizer or optimization parameters.
- etc...


## Results 
### MLP-based model with different features.

| Features  | WER (%) |
| --------- | ------- |
| Best DNN model from [1]  | 27.88  |
| Best DNN model from [2]  | 31.0  |
| fmllr  | 34.06  |
| fbank  | 33.88  |
| fbank + pitch | 33.91  |
| mfcc + pitch  | 34.39  |
| fbank + pitch + fmllr  | **32.74**  |

- Given that this model is a standard MLP with no extra fancy features, don't expect to obtain better results than state-of-the-art models. It's best to be used as a sample for building baseline models.
- As you can see the WER does not reach as low as the model from [1], but it may be worth noting researchers from that paper used a more sophisticated methods of data augmentation by modifying tempo at the phoneme level.
- I was able to get competitive results with [2] which used fbank+pitch features in a novel GNN-based acoustic model. But it's probably not a fair compraison since they did not use data augmentation.


### References
1. F. Xiong, J. Barker, and H. Christensen, "Phonetic Analysis of Dysarthric Speech Tempo and Applications to Robust Personalised Dysarthric Speech Recognition," in IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP), Brighton, UK, May 2019
2. Liu, S., Hu, S., Liu, X., & Meng, H. (2019). On the Use of Pitch Features for Disordered Speech Recognition. Proc. Interspeech 2019, 4130-4134.
