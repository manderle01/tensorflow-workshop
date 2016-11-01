# Train on the cloud: distributed

With Cloud ML, it is nearly as easy to run your TensorFlow model training jobs on multiple workers as on a single worker.

Although the TensorFlow MNIST example that we have been using only allows training on a single worker, we have updated it to allow distributed training in the way that is advised on TensorFlow's Distributed TensorFlow how-to.

We have also updated the example to support the Cloud ML (online and batch) prediction services, although we will save deploying a model to the prediction services for the Prediction Quickstart.

You can find the updated code here:

cd mnist/distributed/
Test the training code locally

First, verify that the model trains locally (on a single worker):

# Clear the output from any previous local run.
rm -rf output/
# Train locally.
python -m trainer.task \
  --train_data_paths=gs://cloud-ml-data/mnist/train.tfr.gz \
  --eval_data_paths=gs://cloud-ml-data/mnist/eval.tfr.gz \
  --output_path=output
You should see output similar to:

INFO:root:Original job data: {}
INFO:tensorflow:global_step/sec: 0
INFO:tensorflow:global_step/sec: 0
INFO:root:Train [master/0], step 1: loss: 2.340, accuracy: 0.090 (0.020 sec) 49.0 global steps/s, 49.0 local steps/s
INFO:root:Eval, step 1: loss: 2.334, accuracy: 0.083
INFO:root:Train [master/0], step 304: loss: 1.990, accuracy: 0.373 (2.293 sec) 133.3 global steps/s, 133.3 local steps/s
INFO:root:Eval, step 304: loss: 1.512, accuracy: 0.612
INFO:root:Train [master/0], step 589: loss: 1.650, accuracy: 0.509 (4.163 sec) 152.4 global steps/s, 152.4 local steps/s
INFO:root:Eval, step 589: loss: 1.095, accuracy: 0.695
INFO:root:Train [master/0], step 870: loss: 1.458, accuracy: 0.568 (6.085 sec) 146.2 global steps/s, 146.2 local steps/s
INFO:root:Eval, step 870: loss: 0.951, accuracy: 0.713
INFO:root:Train [master/0], step 1163: loss: 1.328, accuracy: 0.604 (7.974 sec) 155.1 global steps/s, 155.1 local steps/s
INFO:root:Eval, step 1163: loss: 0.875, accuracy: 0.733
INFO:root:Train [master/0], step 1481: loss: 1.234, accuracy: 0.631 (9.888 sec) 166.1 global steps/s, 166.1 local steps/s
INFO:root:Eval, step 1481: loss: 0.821, accuracy: 0.765
INFO:root:Adjusting eval interval from 1.00s to 1.24s
...
INFO:root:Train [master/0], step 4859: loss: 0.821, accuracy: 0.780 (29.072 sec) 166.9 global steps/s, 166.9 local steps/s
INFO:root:Train [master/0], step 4938: loss: 0.815, accuracy: 0.782 (29.314 sec) 327.4 global steps/s, 327.4 local steps/s
INFO:root:Eval, step 4938: loss: 0.457, accuracy: 0.908
INFO:root:Exporting prediction graph to output/model
INFO:root:Final metrics after 5000 steps, loss: 0.456, accuracy: 0.909
Submit the training job

Choose a name for your training job, e.g. "mnist_distributed_yourusername". This should start with a letter and contain only letters, numbers and underscores.

JOB_NAME=<your job name>
Clear the output from any previous cloud run:

PROJECT_ID=`gcloud config list project --format "value(core.project)"`
TRAIN_BUCKET=gs://${PROJECT_ID}-ml
TRAIN_PATH=${TRAIN_BUCKET}/${JOB_NAME}
gsutil rm -rf ${TRAIN_PATH}
Create a simple configuration file that specifies the Cloud ML STANDARD_1 scale tier, i.e. many workers and a few parameter servers:

cat << EOF > config.yaml
trainingInput:
  # Use a cluster with many workers and a few parameter servers.
  scaleTier: STANDARD_1
EOF
Finally, submit the training job:

gcloud beta ml jobs submit training ${JOB_NAME} \
  --package-path=trainer \
  --module-name=trainer.task \
  --staging-bucket="${TRAIN_BUCKET}" \
  --region=us-central1 \
  --config=config.yaml \
  -- \
  --train_data_paths="gs://cloud-ml-data/mnist/train.tfr.gz" \
  --eval_data_paths="gs://cloud-ml-data/mnist/eval.tfr.gz" \
  --output_path="${TRAIN_PATH}/output"
This command is identical to the one that we used above to train on a single worker, except for:

addition of the scale tier via the config file
modification of the training code's command-line flags to account for the new code version
Wait for the training job to finish

Note: This process is identical to the one that we used above to train on a single worker.
Check the progress of the job and wait for it to finish:

gcloud beta ml jobs describe --project ${PROJECT_ID} ${JOB_NAME}
You should see state: SUCCEEDED once the job completes.

Alternatively, you can check the progress in Machine Learning > Jobs on Cloud Platform Console.

Inspect the output

In this sample, outputs are saved to ${TRAIN_PATH}/output; to list them, run:

gsutil ls ${TRAIN_PATH}/output
Inspect the Stackdriver logs

You can use Stackdriver Logging to verify that the model trained on multiple workers.

The easiest way to find the logs for your job is to select your job in Machine Learning > Jobs on Cloud Platform Console, and then click "View logs".

Click on master-replica-0 in the "All logs" drop down; you should see logs similar to:

Train [master/0], step 0: loss: 2.378, accuracy: 0.080 (0.030 sec) 0.0 global steps/s, 33.5 local steps/s
Eval, step 0: loss: 2.354, accuracy: 0.079
Train [master/0], step 104: loss: 2.099, accuracy: 0.305 (7.484 sec) 14.0 global steps/s, 14.0 local steps/s
Eval, step 104: loss: 1.808, accuracy: 0.562
Adjusting eval interval from 1.00s to 5.26s
Train [master/0], step 204: loss: 1.827, accuracy: 0.477 (13.752 sec) 16.0 global steps/s, 16.0 local steps/s
Train [master/0], step 317: loss: 1.553, accuracy: 0.585 (14.760 sec) 112.1 global steps/s, 112.1 local steps/s
Train [master/0], step 493: loss: 0.704, accuracy: 0.831 (15.762 sec) 175.6 global steps/s, 104.7 local steps/s
Train [master/0], step 699: loss: 0.609, accuracy: 0.849 (16.777 sec) 203.0 global steps/s, 99.5 local steps/s
Train [master/0], step 935: loss: 0.551, accuracy: 0.859 (17.779 sec) 235.6 global steps/s, 115.8 local steps/s
Train [master/0], step 990: loss: 0.541, accuracy: 0.861 (18.016 sec) 232.0 global steps/s, 113.9 local steps/s
Eval, step 990: loss: 0.400, accuracy: 0.890
Adjusting eval interval from 5.26s to 5.32s
Train [master/0], step 1829: loss: 0.445, accuracy: 0.881 (24.347 sec) 132.5 global steps/s, 16.7 local steps/s
Train [master/0], step 2116: loss: 0.333, accuracy: 0.906 (25.356 sec) 284.5 global steps/s, 100.1 local steps/s
Train [master/0], step 2416: loss: 0.326, accuracy: 0.909 (26.366 sec) 297.0 global steps/s, 98.0 local steps/s
Train [master/0], step 2743: loss: 0.281, accuracy: 0.924 (27.376 sec) 323.7 global steps/s, 103.9 local steps/s
Train [master/0], step 3175: loss: 0.287, accuracy: 0.916 (28.383 sec) 429.0 global steps/s, 109.2 local steps/s
Train [master/0], step 3291: loss: 0.286, accuracy: 0.917 (28.660 sec) 419.4 global steps/s, 108.5 local steps/s
Eval, step 3291: loss: 0.264, accuracy: 0.926
Exporting prediction graph to /tmp/0d3879fb-f159-4695-89ff-8616492ac24c/model
Final metrics after 5006 steps, loss: 0.228, accuracy: 0.935
Now choose worker-replica-0 instead of master-replica-0; you should see logs similar to:

Train [worker/0], step 348: loss: 0.729, accuracy: 0.734 (0.034 sec) 10316.6 global steps/s, 29.6 local steps/s
Train [worker/0], step 549: loss: 0.677, accuracy: 0.834 (1.037 sec) 200.4 global steps/s, 99.7 local steps/s
Train [worker/0], step 762: loss: 0.587, accuracy: 0.851 (2.044 sec) 211.4 global steps/s, 109.2 local steps/s
Train [worker/0], step 997: loss: 0.540, accuracy: 0.861 (3.047 sec) 234.5 global steps/s, 119.7 local steps/s
Train [worker/0], step 1123: loss: 0.521, accuracy: 0.865 (4.053 sec) 125.2 global steps/s, 125.2 local steps/s
Train [worker/0], step 1239: loss: 0.501, accuracy: 0.869 (5.054 sec) 115.9 global steps/s, 115.9 local steps/s
Train [worker/0], step 1356: loss: 0.488, accuracy: 0.872 (6.057 sec) 116.7 global steps/s, 116.7 local steps/s
Train [worker/0], step 1473: loss: 0.476, accuracy: 0.874 (7.065 sec) 116.0 global steps/s, 116.0 local steps/s
Train [worker/0], step 1590: loss: 0.465, accuracy: 0.876 (8.065 sec) 117.0 global steps/s, 117.0 local steps/s
Train [worker/0], step 1776: loss: 0.450, accuracy: 0.880 (9.076 sec) 184.0 global steps/s, 106.8 local steps/s
Train [worker/0], step 2042: loss: 0.329, accuracy: 0.908 (10.077 sec) 265.9 global steps/s, 101.0 local steps/s
Train [worker/0], step 2339: loss: 0.328, accuracy: 0.909 (11.082 sec) 295.4 global steps/s, 98.5 local steps/s
Train [worker/0], step 2647: loss: 0.320, accuracy: 0.909 (12.085 sec) 306.9 global steps/s, 103.6 local steps/s
Train [worker/0], step 3057: loss: 0.288, accuracy: 0.921 (13.090 sec) 408.3 global steps/s, 107.6 local steps/s
Train [worker/0], step 3454: loss: 0.271, accuracy: 0.959 (14.093 sec) 395.8 global steps/s, 113.7 local steps/s
Train [worker/0], step 3898: loss: 0.254, accuracy: 0.932 (15.100 sec) 440.8 global steps/s, 117.2 local steps/s
Train [worker/0], step 4341: loss: 0.254, accuracy: 0.929 (16.101 sec) 442.3 global steps/s, 121.8 local steps/s
Train [worker/0], step 4782: loss: 0.251, accuracy: 0.929 (17.110 sec) 437.4 global steps/s, 116.0 local steps/s
Alternatively, you can read the logs on the command-line as described above.

Inspect the summary logs

Like when training on a single worker (or locally), you can inspect the behavior of your training by launching TensorBoard and pointing it at the summary logs produced during training — both during and after execution.

This TensorBoard looks very similar to the previous TensorBoards. If you would like to run TensorBoard for this example, see the previous instructions and set --logdir=${TRAIN_PATH}/output.