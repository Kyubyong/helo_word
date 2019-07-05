# A Neural Grammatical Error Correction System Built On Better Pre-training and Sequential Transfer Learning
Yo Joong Choe, Jiyeon Ham, Kyubyong Park, Yeoil Yoon
https://arxiv.org/abs/1907.01256

## Dependencies

- fairseq
- pattern3 (cp tree.py /opt/conda/envs/pytorch-py3.6/lib/python3.6/site-packages/pattern3/text/)
- errant
- tqdm
- hunspell
- spacy==1.9.0
- nltk
- gdown

## Preprocess
```
python preprocess.py
```

## Restricted Track
- Prepare the data
```
python gec/prepare.py --track 1
```
- Pre-train
  - If you train the model, the system will automatically create a checkpoint directory.
  - Fill it in {ckpt_dir}.
```
python gec/train.py --track 1 --train-mode pretrain --model base
python gec/evaluate.py --track 1 --subset valid --ckpt-dir {ckpt_dir}
```
- Train
  - If you evaluate the model, the system will automatically create an output directory.
  - Fill the previous model output directory into {prev_model_output_dir}.
```
python gec/train.py --track 1 --train-mode train --model base --lr 1e-4 --max-epoch 40 --reset \
  --prev-model-output-dir {prev_model_output_dir}
python gec/evaluate.py --track 1 --subset valid --ckpt-dir {ckpt_dir}
```
- Fine-tune
  - Fill the best validation report into {prev_model_output_fpath}.
  - Then `error_type_contro.py` will give you a list of error types to be removed.
  - Fill them into {remove_error_type_lst}.
```
python gec/train.py --track 1 --train-mode finetune --model base --lr 5e-5 --max-epoch 80 --reset \
  --prev-model-output-dir {prev_model_output_dir}
python gec/evaluate.py --track 1 --subset valid --ckpt-dir {ckpt_dir}
python gec/error_type_control.py --report {prev_model_output_fpath} --max_error_types 10 --n_simulations 1000000
python gec/evaluate.py --track 1 --subset test --ckpt-fpath {ckpt_fpath} --remove-unk-edits \
  --remove-error-type-lst {remove_error_type_lst} \
  --apply-rerank --preserve-spell --max-edits 7 
```

## Low Resource Track
- Prepare the data
```
python gec/prepare.py --track 3
```
- Pre-train
```
python gec/train.py --track 3 --train-mode pretrain --model base
python gec/evaluate.py --track 3 --subset valid --ckpt-dir {ckpt_dir}
```
- Train
```
python gec/train.py --track 3 --train-mode finetune --model base --max-epoch 40 \
  --prev-model-output-dir {prev_model_output_dir} 
python gec/evaluate.py --track 3 --subset valid --ckpt-dir {ckpt_dir}
python gec/evaluate.py --track 3 --subset test --ckpt-fpath {ckpt_fpath} --remove-unk-edits \
  --remove-error-type-lst {remove_error_type_lst} \
  --apply-rerank --preserve-spell --max-edits 7 
```