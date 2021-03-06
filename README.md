# eagletrt-telemetria-exporter

A local webapp made with **VueJs** and **NodeJs** to easily export the telemetry data of **[@eagleTRT](https://github.com/eagletrt)**.

## Export database data as json

<br />
<p align="center">
  <img src="https://github.com/euberdeveloper/eagletrt-telemetria-exporter/raw/master/docs/assets/demo_json.gif">
</p>
<br />

## Export database data as csv

<br />
<p align="center">
  <img src="https://github.com/euberdeveloper/eagletrt-telemetria-exporter/raw/master/docs/assets/demo_csv.gif">
</p>
<br />

## Fetching schema error

<br />
<p align="center">
  <img src="https://github.com/euberdeveloper/eagletrt-telemetria-exporter/raw/master/docs/assets/fetching_database_error.png">
</p>
<br />

## Exporting error

<br />
<p align="center">
  <img src="https://github.com/euberdeveloper/eagletrt-telemetria-exporter/raw/master/docs/assets/exporting_error.png">
</p>
<br />

## Server startup

<br />
<p align="center">
  <img src="https://github.com/euberdeveloper/eagletrt-telemetria-exporter/raw/master/docs/assets/server_start_log.gif">
</p>
<br />

## Server exporting log

<br />
<p align="center">
  <img src="https://github.com/euberdeveloper/eagletrt-telemetria-exporter/raw/master/docs/assets/server_export_log.gif">
</p>
<br />

## Project purpose

This project was made for **[@eagleTRT](https://github.com/eagletrt)**. The telemetry data are saved as a **MongoDB** in a **Raspberry** inside the car. The purpose of this project is giving a fast way to export the data as json or csv. The project is a **NodeJs** server running in the raspberry and serving a **VueJs** webapp. It should be used with desktop devices, in that mobile support was not needed.

## User usage

To see a demo, scroll up to the top of this README.

Usage:

1. The user types in a browser `https://IP:PORT`, where `IP` is the ip of the raspberry and `PORT` is the port of the server.
2. The webapp asks the server for the **database schema** and shows it.
3. Three columns are shown. The first shows the collections. When the user select a collectoin, in the second column appear the sessions of that collection. The user can select or unselect sessions by clicking on them. All the selected sessions appear in the third column, organized by collection. The selected sessions can be unselected also by clicking them in the third column.
4. Once selected the sessions to export, the user clicks the **JSON** or the **CSV** button, depending on the desidered format.
5. The webapp sends the request to the server and wait for a **zipped** file of the exported sessions.
6. After the server answers to the webapp, the zip file named with a human-readable timestamp is **downloaded**. Then the webapp comes back to point `2`.

**Note:** It is possible that the browser shows an alert because the site is unsercure. You can just ignore it.

## Developer usage

To put the exporter in the raspberry:

1. Connect the raspberry to the internet.
2. Install NodeJs if it is not installed.
3. Clone the repository.
4. Open the terminal in the root directory of the repository.
5. Execute `npm i`.
6. Execute `npm run start`.

To change the server port or the mongodb uri:

1. Open the file `config.json`.
2. Set the property `PORT` as needed.
3. Set the property of the object`MONGO` as needed.

To change something in the frontend:

1. Open the terminal in the directory `vuejs` of the repository.
2. Execute `npm i`.
3. Make changes to the frontend source code.
4. See changes by executing `npm run serve` and open a browser in `https://IP:8080`. Start the server if backend api are needed.
5. Open the terminal in the root directory of the repository.
6. Execute `npm run build:frontend`.
7. Execute `npm run start` to start the server.

To change the api hostname of the frontend:

1. Open the file `vuejs/src/config.json` of this repository.
2. Change the `host` and the `port` properties as needed.
3. Open the terminal in the root directory of this repository.
4. Execute `npm run build:frontend`.
5. Execute `npm run start` to start the server.

Note: If you can not modify `vuejs/src/config.json`, you can pass url parameters to the webapp, such as `https://IP:PORT?hostname=localhost:2323` or `https://IP:PORT?host=localhost&port=2323`.

## Automatize with SystemCtl

On the Raspberry runs Linux Ubuntu. This means that **systemctl** is available.

To make the exporter run on boot:

1. Open a terminal.
2. Go to the `/etc/systemd/system` folder.
3. Create a file by executing `touch exporter.service`.
4. Copy this code into that file.
```
[Unit]
Description=Eagle-TRT telemetry exporter webapp
Wants=mongodb.service
After=mongodb.service
StartLimitIntervalSec=0
[Service]
Type=simple
Restart=always
RestartSec=5
User=ubuntu
ExecStart=/usr/bin/node /home/ubuntu/eagletrt-telemetria-exporter/main.js
WorkingDirectory=/home/ubuntu/eagletrt-telemetria-exporter
[Install]
WantedBy=multi-user.target
```
5. Execute `sudo systemctl daemon-reload` to make the changes effective.
6. Execute `sudo systemctl start exporter`. The exporter should start running at this step.
7. Execute `sudo systemctl enable exporter` to make the exporter start on boot.

## Versions

All the gif and images refers to the *1.0* version. Currently, on master, there is the *2.1* version. The difference between the *1.0* and the *2.0* is that the new versions support the *new database structure*, but the usage and the ui is still the same.

### Version 2.1

The webapp is now served via https, with a self-signed certificate. This was introduced after in the new version of most browsers all non-https requests worked only with localhost. The webapp could be warned as unsafe, but it is possible to simply skip the warning and use it.

### Version 2.0

The structure is organized in **databases**, **collections** and **sessions**:

* The database is always the same: `eagletrt`
* The collection can be arbitrarily chosen. It can be for instance the name of the car (`chimera` or `fenice`) or `test`
* The session is determined by the telemetry of the car. It is a document inside a collection, containing the fields representing the timestamp when the session was created, the pilot who was driving the car and the type of race he was performing. The combination of the three is the **sessionName**. A collection can contain several **session documents**, along with their **record documents**, that have have the same **sessionName** value of the session in which they were saved

When exporting:

* __as json__: For each database will be create a folder. Each database folder will contain a json file for each exported collection. Each of these json files will contain both the session documents and the record documents of the exported session. This is because the main usage of the json export is to import the data in another mongodb and to achive this all the sessions of a collection need to stay in an unique json file
* __as csv__: For each database will be create a folder. Each database folder will contain a folder for each exported collection. Each collection folder will contain a folder for each exported session. Each session folder will contain the organized-per-property csv files.

### Version 1.0

The structure is organized in **databases** and **collections**: 

* The database can be for instance the name of the car whose data has been saved, such as `chimera` or `fenice`. 
* The collection is the combination of the timestamp when the telemetry created that collection, the pilot who drove the car and the type of race. All the documents in the collection represents the data saved in 500ms.

When exporting:

* __as json__: For each database will be create a folder. Each database folder will contain a json file for each exported collection.
* __as csv__: For each database will be create a folder. Each database folder will contain a folder for each exported collection. Each collection folder will contain the organized-per-property csv files.
