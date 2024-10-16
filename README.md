# ABC DevSecOps Student Project
This project was created as part of the [ABC DevSecOps](https://www.abcdevsecops.pl/) course. The course is designed to help participants integrate security practices seamlessly into their DevOps pipeline, focusing on secure coding, vulnerability scanning, and implementing CI/CD best practices.

# About the Course
The ABC DevSecOps course, provided by Bezpieczny Kod in collaboration with BRAVE Courses, aims to teach the following key skills:

- **Security in CI/CD Pipelines**: Learn to integrate security tools like ZAP, OSV-Scanner, TruffleHog, and Semgrep into the continuous integration and deployment workflows.
- **Automation of Security Scans**: Use automated tools to catch vulnerabilities early in development, enabling developers to quickly address security issues without disrupting workflows.
- **DevSecOps Practices**: Understand best practices for balancing development speed with application security in DevOps environments.

# Repository Overview
This repository includes:

- **Juice-Shop App**: The official sources of the Juice-Shop project, which is known for its security vulnerabilities. This serves as a testbed for security scanning tools.
- **Security Tools**: Resources for scanners such as:
  - **ZAP Passive Scan**: Utilize both passive and active ZAP scans to detect vulnerabilities, with passive scans identifying issues without attacks, and active scans simulating attacks for deeper analysis.
- **Jenkins Pipeline**: The Jenkinsfile contains the pipeline configurations. As of now, it includes stages for ZAP passive scanning and OSV-Scanner checks. Further stages will be added as I progress through the course.
