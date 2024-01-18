# fuel50.tf

provider "aws" {
  region = "us-east-1" 
}

resource "aws_vpc" "fuel50" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "fuel50" {
  vpc_id                  = aws_vpc.fuel50.id
  cidr_block              = "10.0.0.0/24"
  availability_zone       = "us-east-1a"
}

resource "aws_security_group" "fuel50" {
  name        = "fuel50_security_group"
  description = "Allow inbound HTTP and HTTPS traffic"

  vpc_id = aws_vpc.fuel50.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web" {
  ami             = "ami-023c11a32b0207432" 
  instance_type   = "t2.micro"
  subnet_id       = aws_subnet.fule50.id
  security_group  = [aws_security_group.fuel50.id]

  user_data = <<-EOF
              #!/bin/bash
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              echo '<!DOCTYPE html><html><head><title>Hello World</title></head><body><div style="text-align: center; padding: 50px;"><h1>Hello World</h1></div></body></html>' > /var/www/html/index.html
              EOF

  tags = {
    Name = "fuel50_instance"
  }
}
