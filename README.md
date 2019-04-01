# FashionAI - Clothe Keypoints Detection

> A keypoint detection scheme __based on OpenPose__.

[![996.icu](https://img.shields.io/badge/link-996.icu-red.svg)](https://996.icu)

[![LICENSE](https://img.shields.io/badge/license-NPL%20(The%20996%20Prohibited%20License)-blue.svg)](https://github.com/996icu/996.ICU/blob/master/LICENSE)

## Introduction

FashionAI是阿里天池组织的一个服饰关键点检测比赛，包括blouse/dress/outwear/skirt/trousers五种类型服饰。具体情况可参见[FashionAI](https://tianchi.aliyun.com/competition/introduction.htm?spm=5176.100066.0.0.6acd33afRiMB54&raceId=231648).

第一赛季rank 37,第二赛季rank 42，比赛已结束。

[2018-07-24]该比赛已开放成为长期比赛：https://tianchi.aliyun.com/getStart/introduction.htm?spm=5176.11165320.5678.1.68a84226w3Hys5&raceId=231670。

整个过程中在focal loss/dsconv/upsampling/stage 方面实验较多，遗憾的是refine net方面没有来得及完成实验，目前还在尝试。

最终结果为单模型4.51%，如果做refine net或者ensemble model或许会有所提高。

This is an experimental project for FashionAI competetion which is developed based on [OpenPose(Cao et al)](https://arxiv.org/abs/1611.08050) and [tf-openpose](https://github.com/ildoonet/tf-pose-estimation).

And for each clothe type (dress / skirt / outwear / trousers / blouse) we trained one model separately taking vgg19 network as initial weights for the first several layers. Basically each model need 2 days to train on a 8-gpu machine.
    
Some results:

_dress_  | _outwear_ | _blouse_ | _skirt_ | _trousers_ 
:-------:|:---------:|:--------:|:-------:|:----------:
<img width="150" src="https://wx2.sinaimg.cn/mw1024/89ef5361ly1fryzwf5m7uj20e80e80wo.jpg"> | <img width="150" src="https://wx2.sinaimg.cn/mw1024/89ef5361ly1fryzuh00acj20e80e8jtf.jpg"> | <img width="150" src="https://wx3.sinaimg.cn/mw1024/89ef5361ly1fryzx6np9jj20e80e8myp.jpg"> | <img width="150" src="https://wx2.sinaimg.cn/mw1024/89ef5361ly1fryztd881wj20990e8t9z.jpg"> | <img width="150" src="https://wx1.sinaimg.cn/mw1024/89ef5361ly1fryzst7gq5j20e80e83zs.jpg">
<img width="150" src="https://wx1.sinaimg.cn/mw1024/89ef5361ly1fryzwif71ej20e80e8myn.jpg"> | <img width="150" src="https://wx4.sinaimg.cn/mw1024/89ef5361ly1fryzu09t1dj20e80e8767.jpg"> | <img width="150" src="https://wx2.sinaimg.cn/mw1024/89ef5361ly1fryzxcdp3wj20e80e875r.jpg"> | <img width="150" src="https://wx3.sinaimg.cn/mw1024/89ef5361ly1fryzti1vplj20e80e8q5i.jpg"> | <img width="150" src="https://wx3.sinaimg.cn/mw1024/89ef5361ly1fryzse0nffj20an0e8acc.jpg">
<img width="150" src="https://wx1.sinaimg.cn/mw1024/89ef5361ly1fryzwodoa4j20an0e80ts.jpg"> | <img width="150" src="https://wx4.sinaimg.cn/mw1024/89ef5361ly1fryzu40mtgj20e80e8tb8.jpg"> | <img width="150" src="https://wx1.sinaimg.cn/mw1024/89ef5361ly1fryzxsnqg9j20e80e8n0k.jpg"> | <img width="150" src="https://wx1.sinaimg.cn/mw1024/89ef5361ly1fryzt3lqxej20e80e8jsx.jpg"> | <img width="150" src="https://wx4.sinaimg.cn/mw1024/89ef5361ly1fryzs6023zj20e80e8aaw.jpg">

## Experiments

My result is based on vgg19, then I did following experiments about backbone contrast.

class   | _vgg19_  | _SE-ResNet50_ | _SE-ResNeXt50_
:------:|:--------:|:-------------:|:--------------:
outwear|4.71|4.37|-
dress|4.45|4.28|-
blouse|4.06|4.01|-
skirt|3.97|3.62|-
trousers|3.91|3.68|4.94(checking whether error)

1. Multi-scale feature paramid is not an effective scheme which gives basically no difference on result;

2. As many people mentioned OHEM may be useful, weighted-heatmaps/focal loss/hard retrain are no-use in my experiments;

3. Coarse detection + align detection actually will ignore many hard keypoints, it needs further research; 

4. In my 6 CPM stage-outputs, the last two stage do the best, but merging their results will give unstable results;

5. Upsampling and heatmap-resize-in-postprocess really boost the result.

6. Do not try too much before the contrast between backbone has been done!!!![IMPORTANT :(]

7. Batchsize is not so important.

8. ......

## Requirement

tensorflow(1.4.1+)

opencv

scikit-image

numpy

python-prctl(strongly recommended)

...

## Training
    
I wrote a shell bash to facilitate the training procedure. To train a model, taking blouse as an example just set the --tag in ./begin_to_train.sh to 'blouse'.

And then run ./begin_to_train.sh. 

This command will start the training progress and the trained checkpoint models are saved under ./models/trained. Next you need to frozen the model to use it do the prediction.

Just run ./begin_to_frozen.sh. 

This command will generate frozen graph under ./tmp . 

Copy it to the corresponding directory at ./models/trained/clothe_type/graph because the test shell will load model from this directory.
    
```shell
>  cd code

>  ./begin_to_train.sh

>  ./begin_to_frozen.sh

>  cp ./tmp/frozen_graph.pb ./models/trained/blouse/graph

```

That's all !
    

## Test & Submit

Watch this: begine_to_test.sh

Here is another shell for test.

First please make sure thers exits the corresponding frozen_graph.pb under "./models/trained/%s" % clothe_type.

Modify the --tag in begin_to_test.sh and run it.

This command will generate 'submit_%s.csv' % clothe_type under ./submit directory.

After test all 5 clothe_types then `python merge.py` will merge all results into one .csv file which is in the submit csv format.

```shell
>  cd code

>  python run.py --modelpath="./tmp/frozen_graph.pb" --imagepath="../data/test_b" --csv="../data/test_b/test.csv" --tag="skirt" --test="submit" --inputsize='368' --resolution="368x368" --scales="[1.0, (0.5,0.25,1.5), (0.5,0.75,1.5), (0.25,0.5,1.5), (0.75,0.5,1.5), (0.5,0.5,1.5)]"

>  python run.py --modelpath="./tmp/frozen_graph.pb" --imagepath="../data/test_b" --csv="../data/test_b/test.csv" --tag="trousers" --test="submit" --inputsize='368' --resolution="368x368" --scales="[1.0, (0.5,0.25,1.5), (0.5,0.75,1.5), (0.25,0.5,1.5), (0.75,0.5,1.5), (0.5,0.5,1.5)]"

>  python run.py --modelpath="./tmp/frozen_graph.pb" --imagepath="../data/test_b" --csv="../data/test_b/test.csv" --tag="outwear" --test="submit" --inputsize='368' --resolution="368x368" --scales="[1.0, (0.5,0.25,1.5), (0.5,0.75,1.5), (0.25,0.5,1.5), (0.75,0.5,1.5), (0.5,0.5,1.5)]"

>  python run.py --modelpath="./tmp/frozen_graph.pb" --imagepath="../data/test_b" --csv="../data/test_b/test.csv" --tag="blouse" --test="submit" --inputsize='368' --resolution="368x368" --scales="[1.0, (0.5,0.25,1.5), (0.5,0.75,1.5), (0.25,0.5,1.5), (0.75,0.5,1.5), (0.5,0.5,1.5)]"

>  python run.py --modelpath="./tmp/frozen_graph.pb" --imagepath="../data/test_b" --csv="../data/test_b/test.csv" --tag="dress" --test="submit" --inputsize='368' --resolution="368x368" --scales="[1.0, (0.5,0.25,1.5), (0.5,0.75,1.5), (0.25,0.5,1.5), (0.75,0.5,1.5), (0.5,0.5,1.5)]"

>  cd ../submit

>  python merge.py 

```

That's all!
    
## Contact

If you have any questions about this project. then send an email to buptmsg@gmail.com to let me know.
