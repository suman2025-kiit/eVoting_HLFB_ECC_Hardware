## eVoting_HLFB_ECC_Hardware#
##Electronics Voting (e-Voting) application using Hyperledger Fabric##

Step-by-step procedure to upload and update the repository on GitHub
Target repository: https://github.com/suman2025-kiit/eVoting_HLFB_ECC_Hardware
This document explains how to upload (first time) and update (subsequent revisions) the complete project ECC-HLFB-eVOTING-FPGA-Validation/ into the above GitHub repository, including screenshots, README, and source code.

A. Prerequisites (local environment)
1.	Install Git and verify:
git --version
2.	Ensure you can authenticate with GitHub using either a Personal Access Token (PAT) for HTTPS, or SSH key-based authentication.
•	PAT (HTTPS): GitHub account → Settings → Developer settings → Personal access tokens.
•	SSH: generate keys and test connectivity.
ssh -T git@github.com

B. Upload the prepared repository (first-time push)
4.	Download and extract the repository ZIP locally (it should contain the folder ECC-HLFB-eVOTING-FPGA-Validation/).
5.	Open a terminal inside the extracted folder:
cd ECC-HLFB-eVOTING-FPGA-Validation
6.	Initialize Git and create the first commit:
git init
git add .
git commit -m "Initial commit: ECC-HLFB-eVOTING FPGA validation (Vivado/Vitis + Fabric v2.2)"
7.	Connect your local folder to the GitHub remote repository.
•	Option 1 (HTTPS):
git remote add origin https://github.com/suman2025-kiit/eVoting_HLFB_ECC_Hardware.git
•	Option 2 (SSH):
git remote add origin git@github.com:suman2025-kiit/eVoting_HLFB_ECC_Hardware.git
8.	Push the commit to GitHub. If your repository uses main:
git branch -M main
git push -u origin main
•	If it uses master:
git branch -M master
git push -u origin master
9.	Verify on GitHub that all directories appear (hardware/, software/, chaincode/) and that README.md renders correctly.

C. Update the repository (future revisions / incremental changes)
10.	Before editing, pull the latest changes:
git pull origin main
11.	Modify required files (RTL, Vitis app, gateway, chaincode, screenshots).
12.	Check changes:
git status
git diff
13.	Stage and commit:
git add .
git commit -m "Update: refined REC reason codes and gateway submission flow"
14.	Push updates:
git push origin main

D. Uploading screenshots and verifying paths in README
15.	Keep screenshots only inside: hardware/screenshots/.
16.	Ensure filenames match README.md exactly: valid_output.png and invalid_output.png.
17.	Commit and push screenshots:
git add hardware/screenshots/valid_output.png hardware/screenshots/invalid_output.png
git commit -m "Add Vivado/Vitis validation screenshots (ACCEPT/REJECT)"
git push origin main

E. Protecting crypto material (must NOT be committed)
18.	Fabric TLS and user keys/certs must be stored locally only in: software/gateway_service/crypto/.
19.	Confirm .gitignore blocks secrets:
software/gateway_service/crypto/
*.pem
*.key
*.crt
20.	Verify that nothing secret is staged:
git status
•	If keys appear, remove them from staging:
git restore --staged software/gateway_service/crypto/*

F. Recommended GitHub Releases for paper reproducibility
21.	After completing a stable version, tag it:
git tag -a v1.0 -m "ECC-HLFB-eVOTING FPGA validation release v1.0"
git push origin v1.0
22.	Create a GitHub Release named v1.0 and include the tag, screenshots, and the commit hash for reproducibility.


G. Final checklist before submission
•	README completeness: install, build, run, expected outputs.
•	No secrets committed: verify crypto/ and PEM/KEY/CRT files are ignored.
•	Build sanity: make works for Vitis app; chaincode compiles.
•	Folder consistency: matches the manuscript and the published structure.

##The overall architecture is shown below for reference## 
<img width="128" height="371" alt="Overall_Structure" src="https://github.com/user-attachments/assets/2135e9b7-e813-4461-8a2c-b29dfb7f0b9f" />



##The output screenshot is showing both Valid and Invalid Voter tracked by our eVoting_HLFB_ECC_Hardware##
<img width="1536" height="1024" alt="Output_Screen_Shot" src="https://github.com/user-attachments/assets/91c58dda-87ca-4433-8feb-24ae74a21be3" />
