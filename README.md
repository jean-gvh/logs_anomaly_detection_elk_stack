# Real Time Logs Anomaly Detection using an ELK stack
This repository explains the implementation of a logs anomaly detection architecture using ElasticSearch and Kibana, as well as a machine learning model. The purpose of this architecture is to analyze anomalies and suspicious behavior in a cluster of machines in near real time.

Nota bene : This repository does not deals with the set up of the cluster of machines, but focus more on the development of the machine learning model and its usage in production. 

Therefore, the following explainations will be articulated in 3 main points : 
* Code structure and workflow behaviour.
* Justification of the choosen machine learning model.
* Relevancy of the predictions obtained.

Before diving into the details, the diagrams below illustrates the overall architecture : 
![image](https://github.com/user-attachments/assets/fbd0ca65-0769-463a-a6e7-21f3d6836c4b)


# 1. Code structure and workflow behaviour.

Each nodes of the cluster has a script (see logs_exportation folder) which is executed periodiacally (every minutes) and does the following actions :
* **Log File Check**: Ensures the specified log files exist before processing.
* **Log Parsing**: Extracts structured data (date, hostname, process, etc.) from log lines using regex.
* **Data Structuring**: Reads log files, parses valid lines, and stores them in a list of dictionaries.
* **Elasticsearch Connection**: Connects to an Elasticsearch instance via a defined URL.
* **Data Indexing**: Sends structured log data to an Elasticsearch index which is node's specific.
* **Verification**: Queries the index to ensure logs were successfully added and displays the first five entries.

Once new logs arrived in the elastic search cluster, the master node download the most recent logs them before processing the new logs and predicting anomalies on those last (see logs_downloading). In a nutshell it does the following actions : 
* **Elasticsearch Connection**: Connects to an Elasticsearch cluster at a specified host.
* **Log Retrieval**: Fetches all logs from the Elasticsearch indices using a match_all query, with optional fields and a default batch size of 1000. Scroll API is used for retrieving large datasets.
* **CSV Export**: Writes the retrieved logs to a CSV file (all_logs.csv), ensuring all specified fields are included. Missing fields are filled with empty values.

After downloading the most recent logs of all cluster nodes, those lasts will be processed (encoding, null values check, etc...) to be in the expected shape for the machine learning model. Finally, the dataset is given to the machine learnijg model to predict potential anomalies/suspicious logs. If any anomalies or suspicious logs are found, an email is sent to the cluster manager (sysadmin) to keep him updated on the cluster health. This whole process is described in the logs_process_and_prediction folder and it basically consists of : 
* Applied one hot encoding : the new logs read from a csv are transformed into a pandas dataframe and one hot encoding is applied on object variables.
* Model loading : The prediction model is loaded into the execution environment to be executed on the encoded data.
* Model Prediction : Predictions are obtained thanks to the model.
* Alerting : if any suspicious logs or logs that may reflect anomalies are unveiled, an email is send to the cluster manager (sysadmin) with the logs description : message, node name, PID.
