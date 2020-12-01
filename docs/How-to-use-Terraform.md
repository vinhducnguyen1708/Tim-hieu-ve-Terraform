# Cài đặt Terraform
*Cài đặt Terraform trên Centos8*

- B1: Cài đặt yum-config-manager để quản lý repositories
```sh
 sudo yum install -y yum-utils
 ```

- B2: Sử dụng yum-config-manager để thêm repository của HashiCorp
```sh
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
```

- B3: Tiến hành cài đặt
```sh
sudo yum -y install terraform
```

- B4: Kiểm tra
```sh
terraform -v
```

# Thực hiện tạo Configuration file trong Terraform
*Thực hiện khởi tạo các resource trên hệ thống Cloud Openstack đã có sẵn với các thông tin sau đây*
```sh
OS_USERNAME=admin
OS_PASSWORD=Welcome123
OS_PROJECT_NAME=admin
OS_USER_DOMAIN_NAME=Default
OS_PROJECT_DOMAIN_NAME=Default
OS_AUTH_URL=http://controller:5000/v3
OS_IDENTITY_API_VERSION=3
```

- B1: Tạo thư mục chứa projects:
```sh
mkdir terraform_openstack && cd $_
```

- B2: Tạo file `provider.tf` với nội dung

```t
provider "openstack" {
  user_name   = "admin"
  tenant_name = "admin"
  password    = "Welcome123"
  auth_url    = "http://192.168.10.31:5000/v3"
  region      = "Hanoi"
  user_domain_name = "Default"
  project_domain_name = "Default"
}
```
 *File này dùng để khai báo thông tin để trỏ tới API của hệ thống Openstack*

 *Với mỗi một cloud provider ta sẽ đều phải khai báo như vậy để xác thực, kết nối được với cloud provider đó*

- B3: Tạo file `create_network.tf` với nội dung:
    ```t
    resource "openstack_networking_network_v2" "network_1" {
    name           = "network_terraform"
    admin_state_up = "true"
    external = "true"
    }

    resource "openstack_networking_subnet_v2" "subnet_1" {
    name       = "subnet_1"
    network_id = "${openstack_networking_network_v2.network_1.id}"
    cidr       = "192.168.199.0/24"
    ip_version = 4
    }
    ```
    *Ở đây ta sẽ thực hiện khởi tạo các resource network trên hệ thống Openstack*

    - `openstack_networking_network_v2`: tên resource

      `network_1`: tên của quá trình tạo resource này


    - Các config của resource:
    
      `name = "network_terraform"`: Tên sẽ đặt cho dải network khởi tạo

      `admin_state_up = "true"`

      `external = "true"` 

- B4: Thực hiện load các file terraform 
```sh
terraform init
```

- B5: Thực hiện chạy plan để xem sẽ có những thay đổi gì trước khi apply:
```sh
terraform plan
```

- B6: Sau khi đã thực hiện chạy plan và xác nhận những thay đổi với hệ thống thì thực hiện deploy
```sh
terraform apply
```

## Sử dụng Modules và khai báo biến

- B1: Tạo đường dẫn như sau:
```sh
mkdir -p /root/terraform_openstack/modules/flavor/
```
- B2: Tạo file `/root/terraform_openstack/modules/flavor/main.tf` với nội dung:
```t
resource "openstack_compute_flavor_v2" "create_vinh_flavor" {
  name  = "vinh-flavor"
  ram   = "512"
  vcpus = "1"
  disk  = "${var.disk_flavor}"
  region = "Hanoi"
}
```
- B3: Thêm vào file `provider.tf` với nội dung
```t
...
module "flavor" {
    source = "/root/terraform_openstack/modules/flavor"
}
```

- B4: Tạo file `/root/terraform_openstack/modules/flavor/variables.tf` với nội dung
```t
variable "disk_flavor" {
  description = "The number of disk to use for flavor"
  default = "10"
}
```

- B5: Thực hiện chạy
```sh
terraform refresh

terraform init

terraform plan

terraform apply
```