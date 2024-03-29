name: CI

on:
  push:

jobs:
  sast_scan:
    name: Run Bandit Scan
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.8

      - name: Install Bandit
        run: pip install bandit

      - name: Run Bandit Scan
        run: bandit -ll -ii -r . -f json -o bandit-report.json

      - name: Upload Bandit Artifact
        uses: actions/upload-artifact@v2
        with:
          name: bandit-findings
          path: bandit-report.json

  image_scan:
    name: Docker Image Scan
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Docker
        uses: docker/setup-buildx-action@v1
        with: 
            docker_version: '20.10.7'

      - name: Pull tharuninfo/erpui:v2-154 Docker Image
        run: docker pull tharuninfo/erpui:v2-154

      - name: Docker Scout Scan
        run: |
          curl -fsSL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh -o install-scout.sh
          sh install-scout.sh
          echo ${{ secrets.REPO_PWD }} | docker login -u ${{ secrets.REPO_USER }} --password-stdin
          docker scout quickview
          docker scout cves
         
