# 🚀 DevOps Project: Deploy Django Application on AWS (ECS + ECR + Fargate)

This project shows how to **containerize** a Django web app with **Docker**, push it to **Amazon ECR**, and deploy it using **ECS Fargate**, all managed with **Terraform**.
It demonstrates real-world **Infrastructure as Code (IaC)** for modern DevOps pipelines.

---

## 🛠️ Tech Stack

* **Django** — Python web framework
* **Docker** — containerize your app
* **Amazon ECR** — private image registry
* **Amazon ECS (Fargate)** — serverless containers
* **Terraform** — provision all AWS resources
* **CloudWatch Logs** — monitor your containers

---

## 📁 Project Structure

```plaintext
project-root/
│
├── django_app/             # Your Django app source code
├── Dockerfile              # Dockerfile for the container image
├── requirements.txt        # Python dependencies
├── manage.py               # Django CLI entry point
├── terraform/              # Terraform IaC configs
└── README.md               # This guide!
```

---

## ✅ Step 1 — Develop & Verify Django Locally

1️⃣ Install Python packages:

```bash
pip install -r requirements.txt
```

2️⃣ Run Django server:

```bash
python manage.py runserver
```

3️⃣ Visit:

```
http://127.0.0.1:8000/hello/
```




---

## ✅ Step 2 — Dockerize the App

**Dockerfile example:**

```dockerfile
FROM python:3.9

RUN apt-get update && apt-get install -y --no-install-recommends && rm -rf /var/lib/apt/lists/*

WORKDIR /usr/src/app

COPY requirements.txt ./
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

**Build & run container locally:**

```bash
docker build -t django-app:1 .

docker run -p 8000:8000 django-app:1
```


---

## ✅ Step 3 — Push Image to ECR

1️⃣ Authenticate:

```bash
aws ecr get-login-password --region us-east-1 | \
docker login --username AWS --password-stdin 127214189082.dkr.ecr.us-east-1.amazonaws.com
```

2️⃣ Tag & push:

```bash
docker tag django-app:1 127214189082.dkr.ecr.us-east-1.amazonaws.com/django-app:1

docker push 127214189082.dkr.ecr.us-east-1.amazonaws.com/django-app:1
```

✅ Confirm image shows up in ECR Console.

---

## ✅ Step 4 — Provision Infrastructure with Terraform

**Key resources:**

* VPC/Subnets
* Security Group (port 8000)
* ECS Cluster
* IAM Role for ECS Task Execution
* CloudWatch Log Group
* ECS Task Definition (uses Fargate)
* ECS Service

---

**Highlights:**
Below is a *sample* block for the ECS Service:

```hcl
resource "aws_ecs_service" "django" {
  name            = "django-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.main.arn
  desired_count   = 1
  launch_type     = "FARGATE"

  force_new_deployment = true

  depends_on = [aws_iam_role_policy_attachment.ecs_task_execution_role_policy]

  network_configuration {
    subnets         = data.aws_subnets.default.ids
    security_groups = [aws_security_group.allow_tls.id]
    assign_public_ip = true
  }
}
```

---

**Run Terraform:**

```bash
terraform init
terraform plan
terraform apply
```

✅ Output will show your ECR repo URL and the ECS Service deploys your task!

---

## ✅ Step 5 — Verify

* Go to **ECS Console → Clusters → django-cluster → Services → django-service**
* Confirm 1 task is running.
* If needed, check logs in **CloudWatch**.
* Test the **Public IP** to hit `:8000/hello/`

---

## 💡 Key Points

✔️ Use `:1` tag → push new image **every update**
✔️ Always `terraform apply` to bump Task Definition revision → ECS uses latest image
✔️ `force_new_deployment` + `create_before_destroy` = safe zero-downtime update
✔️ Use CloudWatch logs for debugging!

---

## 🎉 Outcome

✅ Containerized Django → Pushed to ECR → Deployed on ECS Fargate → Fully automated with Terraform.

---


## 📬 Connect

Feel free to ⭐️ this repo and reach out for any improvements!
