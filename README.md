Setting up Box
- https://developer.box.com/ -> My Apps -> Create New App (Name it what ever you'd like)

Congifuration
- Generate Developer Token (ONLY GOOD FOR ONE HOUR)
- App Access Level: App + Enterprise Access
- CORS Domains: Add these: http://localhost:3000/, http://localhost:3000, http://localhost (Keep them comma seperated)
- To find the folder id you will need to go to the box website find the folder you want inspect element find the parent of the folder and it should have a data attribute named data-resin-folder_id

Using
- Clone the github repo
- In terminal use the command npm install
- In terminal use the command npm run dev
- Once it opens go to localhost:3000 type in your folder id and your api key and click fetch. This should return you two tables, one with tasks data, and one with comments data.
