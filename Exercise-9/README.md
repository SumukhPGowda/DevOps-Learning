# Exercise 9 - Jenkins Multi-Stage Pipeline

## Objective
Automate the build, test, deployment, and execution of a Python Flask application using Jenkins.

## Files
- app.py: Flask application
- requirements.txt: Python dependencies
- test_app.py: Unit test for the home endpoint
- Jenkinsfile: Declarative Jenkins pipeline

## Pipeline Stages
1. Build: Check Python and pip, then install dependencies.
2. Test: Run unit tests.
3. Deploy: Copy application files into python-app-deploy.
4. Run Application: Start the Flask application in the background.
5. Test Application: Run the unit tests again.

## Expected Result
The tests pass and Jenkins reports:
Pipeline completed successfully!
