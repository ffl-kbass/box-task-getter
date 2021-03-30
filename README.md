Getting Started
- https://developer.box.com/
- My Apps
- Create New App (Name it whatever)

Congifuration
- Generate Developer Token (ONLY GOOD FOR ONE HOUR)
- App Access Level - App + Enterprise Access
- CORS Domains - Add these: http://localhost:3000/, http://localhost:3000, http://localhost

Clone
- Clone this github repo
- Create a .env file in the root directory. In this file make two variables: FOLDER_ID, API_KEY.
- To get the folder id open the dev tools and find the data attribute data-resin-folder_id (It is the parent container for the folder)
- run npm install
- run npm run dev

//REQUIRES NODE and NPM
