# httppost-counter-service
### STEPS
- Create an EC2 Ubuntu 22.04 instance for the counter-service app
- Define network rules in security group
- Install docker, AWS CLI and configure `aws configure`
- Retrieve an authentication token and authenticate your Docker client to ECR registry: `aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com`
- Push code to Github, rajani-patch-1
- Create ECR registry in AWS , name should be same as Github repo HTTPPost-counter-service
-  Write GitHub Actions for CI (build the image and push it to ECR)
-  Create a new release:
        - Fetch all tags.
        - Get the latest tag, assume semver, and sort.
        - If there’s no tag yet, start with v0.0.0.
        - Increment the patch version.
        - Output the next version.
        - Create the release.
- Build a Docker image:
        - Configure AWS credentials.
        - Login to Amazon ECR.
        - Extract the repository name.
        - Build a Docker image.
        - Push the image to ECR.
-   Following secrets are configured :
        - GITHUB_TOKEN: ${{ secrets.ACCESS_TOKEN_2 }}
        - aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        - aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
- Push the GitHub Actions code to the rajani-patch-1 branch and see if the image will be delivered to ECR by executing GitHub Actions.
- Add the Sonar Cloud and Snyk tests:
        - Login with your GitHub account to Sonar Cloud.
        - Create an Organization (optionally).
        - Press + on the right upper corner, then press Analyze new project.
        - Search for your project, select it, and press Set Up.
- Create a new SONAR_TOKEN secret, copy the SonarCloud step to your GitHub Actions yml file, and create a sonar-project.properties file.
- Make sure the Automatic Analysis is off in Administration/Analysis Method

- **Snyk:**

- Login to https://snyk.io/ with GitHub.
- Press “Add project ”on the right upper corner > GitHub, choose your repository, and press “Add selected repository.”
- configure SNYK_TOKEN for code vulnerabilities  https://docs.snyk.io/snyk-api/revoking-and-regenerating-snyk-api-tokens
  
- Write GitHub Actions for CD (pull the image from the ECR to the EC2 instance and run it).
      - save EC2_PEM_KEY and EC2_HOST as secrets to the Github repository and copy compose.yaml from the GitHub repository to the EC2 server.
  
The CI workflow runs `docker compose -f compose.yaml up -d` on the EC2 instance with `HOST_PORT=80`, `ECR_REGISTRY` and `IMAGE_TAG` set, so the app is reachable on port 80.

Counter data is stored in the named volume `counter-data` mounted at `/data`. The image creates `/data` owned by the non-privileged `appuser`, so no `chmod 777` is required. To reset the counter, run `docker compose down -v`.
Push the changes to rajani-patch-1 and verify CI/CD works.
- open the browser and check : `http:<ec2-public-ip>:80`
- try to send a POST request to the app: `curl -X POST http://<ec2-public-ip>:80`
