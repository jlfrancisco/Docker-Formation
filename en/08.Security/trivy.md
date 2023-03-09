In this exercise, you will use the *Trivy* vulnerability scanner to scan an image of your choice.

You can find more information about this project in its Github repository: [https://github.com/aquasecurity/trivy](https://github.com/aquasecurity/trivy)

## Installation

Install *Trivy* using the following command:

```
$ curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/master/contrib/install.sh | sh -s -- -b /usr/local/bin
```

## Scanning an image

Simply run Trivy on an image of your choice

```
$ trivy IMAGE_NAME
```

Example of scanning the image *nginx:1.14*

![Scanning Nginx 1.14](./images/cve-1.png)

You will then get the list of vulnerabilities that have been detected in this image.
Each of these vulnerabilities has a reference that will allow you to know more about it on the site [https://cve.mitre/org](https://cve.mitre.org)

Information obtained concerning the vulnerability *CVE-2020-3810* detected in the image *nginx:1.14*.

![CVE-2020-3810](./images/cve-2.png)