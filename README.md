
# DNN-based ASR with UAspeech
- Baseline script for UAspeech data using pytorch-kaldi based DNN's
- This is just an example on how to use the pytorch-kaldi library to improve the WER of dysarthric speech ASR.
- To use this toolkit you must already have alignments which can be generated from a GMM-HMM acoustic model with kaldi, check the following repositories if you have not already done so.
  - https://github.com/ffxiong/uaspeech
  - https://github.com/abnerLing/dysarthria-asr
  
- The results shown are from my own personal experiments using the typical training methods as previous studies.
  - Speaker dependent models 
  - Training with all healthy speaker utterancse wile only using B1+B3 utterances for dysarthric speakers.
  - The dev set was composed of only 2 speakers from the healthy set.
  - Decoding with the B2 utterances from dysarthric speakers.
### Instructions
1. Install kaldi https://kaldi-asr.org/doc/install.html
2. Install pytorch https://pytorch.org/get-started/locally/

3. clone pytorch-kaldi repository
```
git clone git clone https://github.com/mravanelli/pytorch-kaldi
```
4. Install other required libraries
```
cd pytorch-kaldi
pip install -r requirements.txt
```
5. Run cfg script 
  - You can just download the cfg files and put them in the cfg directory from pytorch-kaldi or clone this repository and run it from there.)
  - I recommend just working in the pytorch-kaldi directory.
``` 
python /pytorch-kaldi/run_exp.py cfg/ua_best.cfg
```
6. Check Results
``` 
cat /pytorch-kaldi/exp/uaspeech_best/decode_test_out_dnn1/scoring_kaldi/best_wer
%WER 32.74 [ 7686 / 23477, 0 ins, 0 del, 7686 sub ] /pytorch-kaldi/exp/uaspeech_best/decode_test_out_dnn1/wer_10_0.0
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
