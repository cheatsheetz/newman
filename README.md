# Newman Cheat Sheet

Newman is the command-line collection runner for Postman. It enables you to run and test Postman collections directly from the command line, making it perfect for continuous integration, automated testing, and bulk testing scenarios.

---

## Table of Contents
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Command Options](#command-options)
- [Environments](#environments)
- [Data Files](#data-files)
- [Reporters](#reporters)
- [CI/CD Integration](#cicd-integration)
- [Custom Scripts](#custom-scripts)
- [Performance Testing](#performance-testing)
- [Docker Usage](#docker-usage)
- [Best Practices](#best-practices)

---

## Installation

### NPM Installation
```bash
# Install Newman globally
npm install -g newman

# Install Newman locally in project
npm install --save-dev newman

# Install with additional reporters
npm install -g newman newman-reporter-html newman-reporter-json

# Check installation
newman --version
```

### Alternative Installation Methods
```bash
# Using Yarn
yarn global add newman

# Using Docker
docker pull postman/newman

# Using Snap (Linux)
sudo snap install newman
```

## Basic Usage

### Running Collections
```bash
# Basic collection run
newman run collection.json

# Run collection from URL
newman run https://api.getpostman.com/collections/collection-id?apikey=your-key

# Run collection with environment
newman run collection.json -e environment.json

# Run collection from Postman Cloud
newman run "https://www.getpostman.com/collections/collection-id" \
  --apikey your-postman-api-key
```

### File Formats Supported
```bash
# JSON collection and environment files
newman run collection.json -e environment.json

# Postman Collection v2.1 format
newman run collection.postman_collection.json

# Postman Environment format  
newman run collection.json -e environment.postman_environment.json
```

## Command Options

### Core Options
| Option | Description | Example |
|--------|-------------|---------|
| `-e, --environment` | Environment file | `-e prod.json` |
| `-d, --iteration-data` | Data file for iterations | `-d users.csv` |
| `-g, --globals` | Global variables file | `-g globals.json` |
| `-n, --iteration-count` | Number of iterations | `-n 10` |
| `--folder` | Run specific folder | `--folder "User Tests"` |
| `--timeout` | Request timeout (ms) | `--timeout 5000` |
| `--delay-request` | Delay between requests (ms) | `--delay-request 1000` |

### Advanced Options
```bash
# Bail on first failure
newman run collection.json --bail

# Suppress exit codes (for CI)
newman run collection.json --suppress-exit-code

# Disable SSL verification
newman run collection.json --insecure

# Set working directory
newman run collection.json --working-dir ./tests

# Verbose output
newman run collection.json --verbose

# Color output
newman run collection.json --color on

# Disable Unicode in output
newman run collection.json --disable-unicode
```

### Request Options
```bash
# Custom timeout settings
newman run collection.json \
  --timeout-request 10000 \
  --timeout-script 5000

# Custom user agent
newman run collection.json \
  --user-agent "Newman/CustomAgent"

# SSL options
newman run collection.json \
  --ssl-client-cert client.pem \
  --ssl-client-key client-key.pem \
  --ssl-ca-cert ca.pem

# Cookie jar
newman run collection.json \
  --cookie-jar cookies.json
```

## Environments

### Using Environment Files
```json
// environment.json
{
  "id": "production-env",
  "name": "Production",
  "values": [
    {
      "key": "base_url",
      "value": "https://api.production.com",
      "enabled": true
    },
    {
      "key": "api_key",
      "value": "prod-api-key-123",
      "enabled": true,
      "type": "secret"
    }
  ]
}
```

```bash
# Run with environment
newman run collection.json -e production.json

# Run with multiple environments (globals + environment)
newman run collection.json -g globals.json -e staging.json

# Override environment variables
newman run collection.json -e production.json \
  --env-var "api_key=override-key-123"
```

### Global Variables
```json
// globals.json
{
  "id": "global-vars",
  "name": "Global Variables",
  "values": [
    {
      "key": "company_name",
      "value": "Acme Corp",
      "enabled": true
    },
    {
      "key": "api_version",
      "value": "v1",
      "enabled": true
    }
  ]
}
```

## Data Files

### CSV Data Files
```csv
// users.csv
username,email,role
john_doe,john@example.com,admin
jane_smith,jane@example.com,user
bob_jones,bob@example.com,moderator
```

### JSON Data Files
```json
// test-data.json
[
  {
    "username": "admin_user",
    "email": "admin@example.com",
    "password": "admin123",
    "role": "admin"
  },
  {
    "username": "test_user",
    "email": "test@example.com",
    "password": "test123",
    "role": "user"
  }
]
```

### Using Data Files
```bash
# Run with CSV data
newman run collection.json -d users.csv

# Run with JSON data
newman run collection.json -d test-data.json

# Specify number of iterations
newman run collection.json -d users.csv -n 3

# Run specific iteration
newman run collection.json -d users.csv --iteration-count 1
```

### Data Variable Usage
```javascript
// In Postman collection, access data variables:
// URL: {{base_url}}/users
// Body: {"username": "{{username}}", "email": "{{email}}"}

// Pre-request Script
console.log('Current iteration data:', pm.iterationData.toObject());

// Test Script  
pm.test("User created with correct data", function () {
    const responseJson = pm.response.json();
    pm.expect(responseJson.username).to.eql(pm.iterationData.get('username'));
});
```

## Reporters

### Built-in Reporters
```bash
# CLI reporter (default)
newman run collection.json --reporters cli

# JSON reporter
newman run collection.json --reporters json \
  --reporter-json-export results.json

# JUnit XML reporter
newman run collection.json --reporters junit \
  --reporter-junit-export results.xml

# Multiple reporters
newman run collection.json --reporters cli,json,junit \
  --reporter-json-export results.json \
  --reporter-junit-export results.xml
```

### External Reporters
```bash
# Install HTML reporter
npm install -g newman-reporter-html

# Use HTML reporter
newman run collection.json --reporters html \
  --reporter-html-export report.html

# Install and use HTMLExtra reporter
npm install -g newman-reporter-htmlextra
newman run collection.json --reporters htmlextra \
  --reporter-htmlextra-export report.html

# Confluence reporter
npm install -g newman-reporter-confluence
newman run collection.json --reporters confluence \
  --reporter-confluence-export report.html

# Slack reporter
npm install -g newman-reporter-slack
newman run collection.json --reporters slack \
  --reporter-slack-webhookurl webhook-url
```

### Custom Reporter Configuration
```bash
# HTML reporter with custom template
newman run collection.json --reporters htmlextra \
  --reporter-htmlextra-export report.html \
  --reporter-htmlextra-template dashboard \
  --reporter-htmlextra-title "API Test Results" \
  --reporter-htmlextra-logs \
  --reporter-htmlextra-darkTheme

# JSON reporter with specific options
newman run collection.json --reporters json \
  --reporter-json-export results.json \
  --reporter-json-export-includeBody
```

## CI/CD Integration

### GitHub Actions
```yaml
name: API Tests
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  api-tests:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '16'
        cache: 'npm'
    
    - name: Install Newman
      run: |
        npm install -g newman
        npm install -g newman-reporter-htmlextra
    
    - name: Run API tests
      run: |
        newman run tests/api-collection.json \
          -e tests/environments/ci.json \
          --reporters htmlextra,cli,junit \
          --reporter-htmlextra-export reports/newman-report.html \
          --reporter-junit-export reports/newman-report.xml \
          --suppress-exit-code
    
    - name: Upload test results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: newman-reports
        path: reports/
    
    - name: Publish test results
      uses: dorny/test-reporter@v1
      if: always()
      with:
        name: Newman Tests
        path: reports/newman-report.xml
        reporter: java-junit
```

### GitLab CI
```yaml
stages:
  - test

api-tests:
  stage: test
  image: node:16
  before_script:
    - npm install -g newman newman-reporter-htmlextra
  script:
    - |
      newman run api-collection.json \
        -e environments/staging.json \
        --reporters htmlextra,cli,junit \
        --reporter-htmlextra-export newman-report.html \
        --reporter-junit-export newman-report.xml \
        --suppress-exit-code
  artifacts:
    reports:
      junit: newman-report.xml
    paths:
      - newman-report.html
    expire_in: 1 week
  only:
    - main
    - develop
```

### Jenkins Pipeline
```groovy
pipeline {
    agent any
    
    tools {
        nodejs '16'
    }
    
    stages {
        stage('Install Newman') {
            steps {
                sh '''
                    npm install -g newman
                    npm install -g newman-reporter-htmlextra
                '''
            }
        }
        
        stage('Run API Tests') {
            steps {
                sh '''
                    newman run api-collection.json \
                      -e environments/production.json \
                      --reporters htmlextra,cli,junit \
                      --reporter-htmlextra-export newman-report.html \
                      --reporter-junit-export newman-report.xml \
                      --suppress-exit-code
                '''
            }
        }
    }
    
    post {
        always {
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: '.',
                reportFiles: 'newman-report.html',
                reportName: 'Newman Test Report'
            ])
            
            junit 'newman-report.xml'
            
            archiveArtifacts artifacts: '*.html,*.xml', allowEmptyArchive: true
        }
    }
}
```

### Azure DevOps
```yaml
trigger:
- main
- develop

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '16.x'
  displayName: 'Install Node.js'

- script: |
    npm install -g newman newman-reporter-htmlextra
  displayName: 'Install Newman'

- script: |
    newman run $(System.DefaultWorkingDirectory)/tests/api-collection.json \
      -e $(System.DefaultWorkingDirectory)/tests/environments/azure.json \
      --reporters htmlextra,cli,junit \
      --reporter-htmlextra-export $(Common.TestResultsDirectory)/newman-report.html \
      --reporter-junit-export $(Common.TestResultsDirectory)/newman-report.xml \
      --suppress-exit-code
  displayName: 'Run Newman tests'

- task: PublishTestResults@2
  inputs:
    testResultsFormat: 'JUnit'
    testResultsFiles: '$(Common.TestResultsDirectory)/newman-report.xml'
    failTaskOnFailedTests: true
  displayName: 'Publish test results'

- task: PublishHtmlReport@1
  inputs:
    reportDir: '$(Common.TestResultsDirectory)'
    reportName: 'Newman HTML Report'
```

## Custom Scripts

### Pre-execution Setup
```javascript
// pre-run-script.js
module.exports = {
    beforeRequest: function(requestName, context, ee, next) {
        console.log(`Executing: ${requestName}`);
        next();
    },
    
    beforeItem: function(err, context, ee, next) {
        // Setup before each request
        console.log('Setting up request context');
        next();
    }
};
```

### Post-execution Processing
```javascript
// post-run-script.js
const fs = require('fs');

module.exports = {
    afterRequest: function(err, context, ee, next) {
        if (err) {
            console.error('Request failed:', err);
        }
        next();
    },
    
    afterCollection: function(err, summary) {
        // Process results
        const results = {
            total: summary.run.stats.requests.total,
            failed: summary.run.stats.requests.failed,
            passed: summary.run.stats.requests.total - summary.run.stats.requests.failed
        };
        
        fs.writeFileSync('test-summary.json', JSON.stringify(results, null, 2));
        console.log('Test Summary:', results);
    }
};
```

### Running with Custom Scripts
```bash
# Newman with custom library
newman run collection.json --newman-library custom-newman-lib.js

# Using Newman programmatically
node run-newman.js
```

### Programmatic Newman Usage
```javascript
// run-newman.js
const newman = require('newman');

newman.run({
    collection: require('./api-collection.json'),
    environment: require('./environment.json'),
    iterationData: './test-data.csv',
    reporters: ['html', 'cli'],
    reporterOptions: {
        html: {
            export: './newman-report.html'
        }
    },
    insecure: true,
    timeout: 180000
}, function (err, summary) {
    if (err) { 
        throw err; 
    }
    
    console.log('Collection run complete!');
    
    // Process summary
    const stats = summary.run.stats;
    console.log(`Total requests: ${stats.requests.total}`);
    console.log(`Failed requests: ${stats.requests.failed}`);
    console.log(`Passed requests: ${stats.requests.total - stats.requests.failed}`);
    
    // Exit with appropriate code
    process.exit(stats.requests.failed > 0 ? 1 : 0);
});
```

## Performance Testing

### Load Testing Configuration
```bash
# Run multiple iterations for load testing
newman run collection.json -n 100 --delay-request 500

# Parallel execution (multiple Newman instances)
for i in {1..5}; do
  newman run collection.json -n 20 &
done
wait

# Stress testing with no delays
newman run collection.json -n 1000 --delay-request 0
```

### Performance Monitoring
```javascript
// In collection tests - monitor response times
pm.test("Response time is acceptable", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});

// Track performance metrics
const responseTime = pm.response.responseTime;
pm.environment.set("avg_response_time", 
    (pm.environment.get("avg_response_time") + responseTime) / 2
);

// Save metrics for analysis
pm.test("Log performance data", function () {
    const perfData = {
        url: pm.request.url,
        method: pm.request.method,
        responseTime: pm.response.responseTime,
        responseSize: pm.response.size,
        timestamp: Date.now()
    };
    
    console.log(JSON.stringify(perfData));
});
```

### Performance Analysis
```bash
# Extract performance data from Newman output
newman run collection.json --reporters cli,json \
  --reporter-json-export results.json

# Process results with custom script
node analyze-performance.js results.json
```

```javascript
// analyze-performance.js
const fs = require('fs');
const results = JSON.parse(fs.readFileSync(process.argv[2]));

const executions = results.run.executions;
const responseTimes = executions.map(exec => exec.response.responseTime);

const stats = {
    avg: responseTimes.reduce((a, b) => a + b) / responseTimes.length,
    min: Math.min(...responseTimes),
    max: Math.max(...responseTimes),
    p95: responseTimes.sort()[Math.floor(responseTimes.length * 0.95)]
};

console.log('Performance Statistics:', stats);
```

## Docker Usage

### Running Newman in Docker
```bash
# Basic Docker run
docker run -t postman/newman run collection.json

# Mount local files
docker run -v $(pwd):/etc/newman -t postman/newman \
  run collection.json -e environment.json

# Docker with custom options
docker run -v $(pwd):/etc/newman -t postman/newman \
  run collection.json \
  -e production.json \
  -d test-data.csv \
  --reporters html,cli \
  --reporter-html-export report.html
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'
services:
  newman:
    image: postman/newman
    volumes:
      - ./tests:/etc/newman
    command: >
      run api-collection.json
      -e production.json
      -d test-data.csv
      --reporters htmlextra,cli,junit
      --reporter-htmlextra-export /etc/newman/report.html
      --reporter-junit-export /etc/newman/results.xml
    depends_on:
      - api-server
  
  api-server:
    image: my-api:latest
    ports:
      - "3000:3000"
```

### Custom Docker Image
```dockerfile
# Dockerfile
FROM postman/newman:alpine

# Install additional reporters
RUN npm install -g newman-reporter-htmlextra newman-reporter-confluence

# Copy test files
COPY tests/ /etc/newman/

# Set working directory
WORKDIR /etc/newman

# Default command
CMD ["run", "api-collection.json", "-e", "docker.json"]
```

## Best Practices

### Collection Organization
```bash
# Use descriptive folder names
API Tests/
├── Authentication/
├── User Management/
├── Data Processing/
└── Integration Tests/

# Naming conventions
newman run "User Management Tests.postman_collection.json"
newman run collection.json --folder "Authentication"
```

### Environment Management
```json
// Use environment-specific configurations
{
  "development": {
    "base_url": "http://localhost:3000",
    "timeout": 5000,
    "debug": true
  },
  "staging": {
    "base_url": "https://staging-api.example.com",
    "timeout": 10000,
    "debug": false
  },
  "production": {
    "base_url": "https://api.example.com", 
    "timeout": 15000,
    "debug": false
  }
}
```

### Error Handling
```bash
# Continue on failures for comprehensive testing
newman run collection.json --suppress-exit-code

# Bail on first failure for fast feedback
newman run collection.json --bail

# Custom timeout handling
newman run collection.json \
  --timeout-request 30000 \
  --timeout-script 10000
```

### Reporting Best Practices
```bash
# Use multiple reporters for different audiences
newman run collection.json \
  --reporters cli,htmlextra,junit \
  --reporter-htmlextra-export detailed-report.html \
  --reporter-junit-export ci-results.xml

# Include logs and request/response data
newman run collection.json \
  --reporters htmlextra \
  --reporter-htmlextra-logs \
  --reporter-htmlextra-export report.html
```

### Security Considerations
```bash
# Don't log sensitive data in CI
newman run collection.json --silent

# Use secure environment variables
export API_KEY="your-secret-key"
newman run collection.json --env-var "api_key=$API_KEY"

# Exclude sensitive collections from version control
echo "*.postman_collection.json" >> .gitignore
```

---

## Resources
- [Newman Official Documentation](https://github.com/postmanlabs/newman)
- [Newman Docker Hub](https://hub.docker.com/r/postman/newman)
- [Newman Reporters](https://github.com/postmanlabs/newman#reporters)
- [Postman Collection SDK](https://github.com/postmanlabs/postman-collection)
- [Newman Examples](https://github.com/postmanlabs/newman/tree/develop/examples)

---
*Originally compiled from various sources. Contributions welcome!*