# tl_detector
##Annotation

followed TensorFlow Object Detection Model Training https://gist.github.com/douglasrizzo/c70e186678f126f1b9005ca83d8bd2ce

- installed labelImg
- annotated with labelImg
- run xml__to_csv.py (link in tutorial gist)

- manually separate raccoon.csv into train.csv and eval.csv csvs.

- run generate_tfrecord.py (retrieve from gist tutorial)
---- be sure to change label map as instructed

python generate_tfrecord.py --csv_input=train.csv  --output_path=train.record
python generate_tfrecord.py --csv_input=eval.csv  --output_path=test.record

## GCP
<pre>
export PROJECT="tl-tf-object-detection"
export YOUR_GCS_BUCKET="tl_tf_object_detection"
curl -H "Authorization: Bearer $(gcloud auth print-access-token)"      https://ml.googleapis.com/v1/projects/${PROJECT}:getConfig
export TPU_ACCOUNT=service-198902660545@cloud-tpu.iam.gserviceaccount.com
gcloud projects add-iam-policy-binding $PROJECT      --member serviceAccount:$TPU_ACCOUNT --role roles/ml.serviceAgent
python object_detection/builders/model_builder_test.py


gsutil -m cp -r train.record gs://${YOUR_GCS_BUCKET}/data/
gsutil -m cp -r test.record gs://${YOUR_GCS_BUCKET}/data/
gsutil cp object_detection/data/traffic_light_label_map.pbtxt gs://${YOUR_GCS_BUCKET}/data/traffic_light_label_map.pbtxt

gsutil cp <<local pipeline.config>>  gs://${YOUR_GCS_BUCKET}/data/pipeline.config

(note one of the files in the next block had to be edited, sed syntax was failing)
bash object_detection/dataset_tools/create_pycocotools_package.sh /tmp/pycocotools
python setup.py sdist
(cd slim && python setup.py sdist)

 gcloud ml-engine jobs submit training `whoami`_object_detection_`date +%s` --job-dir=gs://${YOUR_GCS_BUCKET}/train --packages dist/object_detection-0.1.tar.gz,slim/dist/slim-0.1.tar.gz,/tmp/pycocotools/pycocotools-2.0.tar.gz --module-name object_detection.model_tpu_main --runtime-version 1.8 --scale-tier BASIC_TPU --region us-central1 -- --model_dir=gs://${YOUR_GCS_BUCKET}/train --tpu_zone us-central1 --pipeline_config_path=gs://${YOUR_GCS_BUCKET}/data/pipeline.config

 gcloud ml-engine jobs submit training `whoami`_object_detection_eval_validation_`date +%s` --job-dir=gs://${YOUR_GCS_BUCKET}/train --packages dist/object_detection-0.1.tar.gz,slim/dist/slim-0.1.tar.gz,/tmp/pycocotools/pycocotools-2.0.tar.gz --module-name object_detection.model_main --runtime-version 1.8 --scale-tier BASIC_GPU --region us-central1 -- --model_dir=gs://${YOUR_GCS_BUCKET}/train --pipeline_config_path=gs://${YOUR_GCS_BUCKET}/data/pipeline.config --checkpoint_dir=gs://${YOUR_GCS_BUCKET}/train
  613  gcloud ml-engine jobs list

  gcloud ml-engine jobs list

  </pre>
