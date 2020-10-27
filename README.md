# Cloud-Computing-final-project
This is the final project for Cloud Computing 2020
## Overview
This project consists the process of building a containerized machine learning prediction model and deploy it in a scalable, and elastic platform.

The elastic scale-up performance is tested via Load Test with Locust, this project starts with 1 container or endpoint and verify 2 or more inference endpoints scale
up to 1K requests per second.

There's also a demo video showing project scaling up to 1k+ requests served by multiple endpoints that scale-up. 

Below is the overall process of this project, which consists creatig a BigQuery Machine Learning model, containerizing this ML model, making a docker and deploying the docker to Kubernetes. 


## Detailed Process

###### 1. Using standard SQL queries to create a Machine Learning model in Google BigQuery.

- create a dataset 

- train and deploy a linear regression model
  
  
  
###### 2. Export the model.

- Create a bucket on GCP

- Export the previously trained BigQuery ML model to the bucket. 
```
bq extract -m bqml_tutorial.iris_model gs://some/gcs/path/iris_model
```



###### 4. Download the model

- create a temporary folder
```
mkdir tmp_dir
```

- download the model from bucket to this temporary folder
```
gsutil cp -r gs://<your_bucket_name>/iris_model tmp_dir

gsutil cp -r gs://bigqueryml-293607/iris_model tmp_dir
```



###### 5. Properly version the model

- create a folder for the desired version (1 in this case)
```
mkdir -p serving_dir/iris_model/1
```

- copy the model from temp folder to current folder 
```
cp -r tmp_dir/iris_model/* serving_dir/iris_model/1
```

- delete the temp folder
```
rm -r tmp_dir
```
  
###### 6. Make docker for the model

- pull the tensorflow/serving from docker hub
```
docker pull tensorflow/serving
```

- run tensorflow/serving, naming the container serving_base
```
docker run -d --name serving_base tensorflow/serving
```

- moving the model into the docker container
```
docker cp serving_dir/iris_model serving_base:/models/iris_model
```
- change the environment varible ENV MODEL_NAME and create an image called iris_serving
```
docker commit --change "ENV MODEL_NAME iris_model" serving_base iris_serving
```
- run the docker image to see if the image can react to given instance proporly.
```
docker run -p 8501:8501 -t iris_serving &

curl -d '{"instances": [{"sepal_length":5.0, "sepal_width":2.0, "petal_length":3.5, "petal_width":1.0}]}' -X POST
```
- kill the container
```
docker kill <container_id>

docker rm <container_id>
```
  
  
  
###### 7. Deploying the container to Kubernetes

-
