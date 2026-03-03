# **Tổng hợp Lệnh AI Engineer (Cheat Sheet \- Cập nhật)**

Đây là bảng tổng hợp các câu lệnh được cập nhật, tập trung vào các tác vụ hàng ngày, đã bổ sung thêm các lệnh nâng cao, công cụ mới (Mamba, uv, lsof, Cloud) từ tài liệu hướng dẫn.

## **🐧 Linux & Giám sát Hệ thống**

Nền tảng cho hầu hết mọi công việc: điều hướng server, quản lý file và quan trọng nhất là theo dõi tài nguyên.

| Lệnh | Mô tả |
| :---- | :---- |
| ls \-lha | Liệt kê file/thư mục (bao gồm file ẩn) với chi tiết (kích thước, quyền). |
| cd \[thư\_mục\] | Thay đổi thư mục. cd .. (lùi lại), cd \~ (về home). |
| pushd \[thư\_mục\] | Đẩy thư mục hiện tại vào stack và chuyển đến thư mục mới. |
| popd | Quay lại thư mục cuối cùng đã đẩy (từ pushd). |
| cp \-r \[nguồn\] \[đích\] | Sao chép file hoặc thư mục (\-r cho thư mục, "recursive"). |
| mv \[nguồn\] \[đích\] | Di chuyển hoặc đổi tên file/thư mục. |
| rm \-r \[file/thư\_mục\] | Xóa file hoặc thư mục (\-r cho thư mục). Cẩn thận khi dùng\! |
| mkdir \-p \[tên\_thư\_mục\] | Tạo thư mục mới (\-p tạo cả thư mục cha nếu chưa tồn tại). |
| ln \-s \[nguồn\] \[đích\] | Tạo một liên kết tượng trưng (symbolic link) từ đích trỏ đến nguồn. |
| ssh \[user\]@\[host\] | Kết nối an toàn đến một server từ xa. |
| chmod \+x \[tên\_script\] | Cấp quyền thực thi cho một file (ví dụ: một script .sh). |
| htop | Giao diện tương tác để theo dõi CPU, RAM, và các tiến trình. |
| nvidia-smi | Lệnh quan trọng nhất: Hiển thị thông tin, mức sử dụng GPU và bộ nhớ VRAM. |
| watch \-n 1 nvidia-smi | Tự động chạy nvidia-smi mỗi giây để theo dõi GPU liên tục. |
| nvtop | Giao diện tương tác (TUI) giống htop để theo dõi chi tiết nhiều GPU cùng lúc. |
| tail \-f \[file\_log\] | Theo dõi nội dung của một file log theo thời gian thực. |
| alias l="ls \-lha" | Tạo một lệnh tắt (alias) cho một lệnh dài. |
| history | Hiển thị lịch sử các lệnh đã gõ. |
| ctrl+r | Tìm kiếm ngược trong lịch sử lệnh. |
| iotop | Theo dõi việc sử dụng I/O (đọc/ghi) đĩa của từng tiến trình. |
| iotop \-oP | Chỉ hiển thị các tiến trình (\-P) đang thực sự thực hiện I/O (\-o). |
| nethogs | Theo dõi việc sử dụng băng thông mạng của từng tiến trình. |
| \[lệnh\] \> log.txt | Chuyển hướng đầu ra chuẩn (stdout) vào một tệp (ghi đè). |
| \[lệnh\] 2\> error.log | Chuyển hướng đầu ra lỗi (stderr) vào một tệp. |

## **🕵️ Tìm kiếm & Xử lý Nâng cao (Agent-style)**

Các lệnh dùng để "lọc nhiễu" và tìm chính xác thứ bạn cần, rất hữu ích khi debug server hoặc script.

### **1\. Tìm kiếm Tiến trình (Process)**

| Lệnh | Mô tả |
| :---- | :---- |
| ps aux | grep \[tên\] | Tìm tất cả tiến trình đang chạy trên server có chứa \[tên\] (ví dụ: vllm). |
| ps aux | grep vllm | grep \-v grep | Tìm tiến trình vllm, và dùng grep \-v (invert match) để loại trừ chính lệnh grep ra khỏi kết quả. |
| kill \-9 \[PID\] | "Giết" (force kill) một tiến trình ngay lập tức bằng Process ID của nó (lấy PID từ ps aux, htop, nvtop). |
| pkill \-f \[tên\_tiến\_trình\] | "Giết" tất cả tiến trình có tên khớp \[tên\_tiến\_trình\] (ví dụ: pkill \-f "main\_train.py"). |
| ps aux | grep python | awk '{print $2}' | xargs kill \-9 | Tìm PID của tất cả tiến trình 'python' và 'giết' chúng. |

### **2\. Tìm kiếm File và Nội dung (File & Content)**

| Lệnh | Mô tả |
| :---- | :---- |
| find . \-name "\*.py" | Tìm tất cả file có đuôi .py trong thư mục hiện tại (.) và các thư mục con (đệ quy). |
| grep \-r "batch\_size" . | (Cực kỳ hữu ích) Tìm kiếm đệ quy (\-r) chuỗi "batch\_size" bên trong nội dung của tất cả các file tại thư mục hiện tại. |
| find . \-name "\*.py" \-exec grep \-H "import torch" {} \+ | Kết hợp find và grep: Tìm tất cả file .py, sau đó chạy grep trên các file đó để tìm dòng "import torch". |
| grep \-rl "old\_text" . | xargs sed \-i 's/old\_text/new\_text/g' | Tìm và thay thế "old\_text" bằng "new\_text" trong tất cả các tệp chứa nó. |

### **3\. Tìm kiếm Tệp đang mở & Cổng mạng (lsof)**

| Lệnh | Mô tả |
| :---- | :---- |
| lsof /path/to/file | Hiển thị tiến trình nào đang mở (khóa) một tệp cụ thể. |
| lsof \-i :8080 | Hiển thị tiến trình nào đang sử dụng cổng mạng 8080\. |
| lsof \-p \[PID\] | Liệt kê tất cả các tệp và kết nối mạng đang được mở bởi một PID cụ thể. |
| lsof \-c python | Liệt kê tất cả các tệp đang được mở bởi các tiến trình có tên 'python'. |
| lsof | grep deleted | Tìm các tiến trình đang giữ các tệp đã bị xóa (nhưng chưa giải phóng dung lượng). |
| lsof /dev/nvidia0 | Tìm tất cả các tiến trình đang sử dụng thiết bị GPU (ví dụ: /dev/nvidia0). |

### **4\. Quản lý Dung lượng (Disk Usage)**

| Lệnh | Mô tả |
| :---- | :---- |
| df \-h | Hiển thị tổng quan dung lượng (đã dùng, trống) của tất cả ổ đĩa (\-h là "human-readable"). |
| du \-sh \* | Tính dung lượng (du) của từng file/thư mục trong thư mục hiện tại. Dùng để tìm xem thư mục nào đang chiếm dung lượng nhất. |

## **🐙 Git (Quản lý Phiên bản)**

Dùng để lưu trữ, quản lý các phiên bản của code và cộng tác với team.

### **1\. Git Cơ bản**

| Lệnh | Mô tả |
| :---- | :---- |
| git clone \[url\] | Tải về một repository (GitHub, GitLab...). |
| git clone \--depth 1 \[url\] | Sao chép nông (shallow clone), chỉ lấy commit mới nhất. |
| git pull | Cập nhật code từ repository từ xa về máy local. |
| git status | Hiển thị trạng thái các file đã thay đổi, file mới. |
| git add \[file\] | Thêm một file vào "staging" (chuẩn bị cho commit). Dùng git add . để thêm tất cả. |
| git commit \-m "\[thông\_điệp\]" | Lưu lại các thay đổi (đã add) vào lịch sử. |
| git push | Đẩy các commit từ local lên repository từ xa. |
| git checkout \-b \[tên\_nhánh\] | Tạo một nhánh (branch) mới và chuyển sang nhánh đó. |
| git checkout \[tên\_nhánh\] | Chuyển đổi giữa các nhánh đã có. |
| git branch | Liệt kê tất cả các nhánh. |
| git merge \[tên\_nhánh\] | Gộp các thay đổi từ một nhánh khác vào nhánh hiện tại. |

### **2\. Git Nâng cao (Phục hồi & Quản lý Lịch sử)**

| Lệnh | Mô tả |
| :---- | :---- |
| git log \--oneline \--graph \--decorate | Xem lịch sử commit dưới dạng rút gọn, có biểu đồ nhánh và tên nhánh/tag. |
| git log \--follow \[file\_path\] | Theo dõi lịch sử của một tệp, ngay cả khi nó đã được đổi tên. |
| git log \-S "function\_name" | Tìm kiếm các commit đã thêm hoặc xóa một chuỗi cụ thể (ví dụ: tên hàm). |
| git reset \--soft HEAD\~1 | Hủy commit cuối cùng, nhưng giữ lại các thay đổi trong file và trong staging. |
| git reset \--mixed HEAD\~1 | Hủy commit cuối cùng, giữ lại thay đổi trong file, nhưng xóa khỏi staging. (Mặc định). |
| git reset \--hard HEAD\~1 | (Cẩn thận\!) Hủy commit cuối cùng và xóa bỏ vĩnh viễn mọi thay đổi của commit đó. |
| git revert HEAD | Tạo một commit mới để hoàn tác các thay đổi của commit trước đó (an toàn cho nhánh public). |
| git reflog | Hiển thị "nhật ký tham chiếu", ghi lại mọi hành động (kể cả reset). Dùng để "cứu" commit đã mất. |
| git stash | Tạm thời cất giấu các thay đổi chưa commit (để chuyển nhánh). |
| git stash pop | Lấy lại các thay đổi đã cất giấu gần nhất. |
| git rebase \-i \[commit\_hash\] | Chỉnh sửa lịch sử commit một cách tương tác (gộp commit, sửa mô tả...). |
| git commit \--fixup \[hash\] | Tạo một commit "fixup" để sửa lỗi cho một commit trước đó. |
| git rebase \-i \--autosquash main | Tự động gộp (squash) các commit fixup vào commit gốc khi rebase. |
| git worktree add ../\<tên\_nhánh\> \<tên\_nhánh\> | Checkout một nhánh khác vào một thư mục riêng biệt mà không cần stash. |
| git clean \-fdx | Xóa tất cả các tệp, thư mục không được theo dõi và bị bỏ qua (để dọn dẹp repo). |

## **🐍 Conda, Mamba & UV (Quản lý Môi trường)**

Tách biệt các project với các phiên bản thư viện khác nhau.

### **1\. Conda & Mamba**

| Lệnh | Mô tả |
| :---- | :---- |
| conda create \-n \[tên\_env\] python=3.10 | Tạo một môi trường mới tên \[tên\_env\] với phiên bản Python 3.10. |
| conda create \--prefix ./venv python=3.10 | Tạo môi trường trong một thư mục cục bộ (di động hơn). |
| mamba create \-n \[tên\_env\] python=3.10 | Tương tự conda create, nhưng nhanh hơn rất nhiều . |
| conda activate \[tên\_env\] | Kích hoạt (bắt đầu sử dụng) một môi trường. |
| conda deactivate | Ngừng kích hoạt môi trường hiện tại, quay về môi trường base. |
| conda env list | Liệt kê tất cả các môi trường bạn đã tạo. |
| conda install \[tpkg\] | Cài đặt một package (ví dụ: conda install pytorch ...). |
| mamba install \[pkg\] | Tương tự conda install, nhưng nhanh hơn . |
| conda env export \> environment.yaml | Xuất toàn bộ package trong môi trường hiện tại ra file yaml. |
| conda env create \-f environment.yaml | Tạo lại một môi trường y hệt từ file yaml. |
| conda env remove \-n \[tên\_env\] | Xóa một môi trường và tất cả package bên trong nó. |

### **2\. UV (Thay thế Pip/Venv cực nhanh)**

| Lệnh | Mô tả |
| :---- | :---- |
| uv pip install \[package\] | Cài đặt một package Python (cực nhanh). |
| uv pip install \-r requirements.txt | Cài đặt từ file requirements.txt (cực nhanh). |
| uv pip freeze \> requirements.txt | Tạo file requirements.txt. |

## **🖥️ Tmux (Quản lý Phiên làm việc)**

Giữ cho tiến trình (như training model) tiếp tục chạy ngay cả khi bạn mất kết nối SSH.  
Prefix mặc định là Ctrl+b. Nhấn Prefix, thả ra, rồi nhấn phím lệnh.

| Phím tắt (Sau Prefix) | Mô tả |
| :---- | :---- |
| tmux new \-s \[tên\_session\] | (Chạy ở terminal thường) Tạo một session tmux mới. |
| tmux ls | (Chạy ở terminal thường) Liệt kê tất cả các session đang chạy. |
| tmux attach \-t \[tên\_session\] | (Chạy ở terminal thường) Kết nối lại vào một session đang chạy. |
| d | Detach (tách ra) khỏi session hiện tại mà không tắt nó. |
| c | Tạo một cửa sổ (window) mới (giống như một tab mới). |
| n / p | Chuyển sang cửa sổ tiếp theo (next) / trước đó (previous). |
| % | Chia đôi cửa sổ hiện tại thành 2 ô theo chiều dọc (trái/phải). |
| " | Chia đôi cửa sổ hiện tại thành 2 ô theo chiều ngang (trên/dưới). |
| \[phím\_mũi\_tên\] | Di chuyển giữa các ô (pane) đã chia. |
| & | Đóng cửa sổ (window) hiện tại. |

## **🚀 Slurm (Quản lý Cluster/HPC)**

Hệ thống quản lý hàng đợi (queue) để "đặt lịch" chạy các job training trên các node có GPU.

### **1\. Lệnh Quản lý Job**

| Lệnh | Mô tả |
| :---- | :---- |
| sbatch \[script.slurm\] | Gửi (submit) một job để chạy (dựa trên file cấu hình script.slurm). |
| squeue \-u \[username\] | Hiển thị trạng thái tất cả các job của bạn (đang chờ: PD, đang chạy: R). |
| scancel \[job\_id\] | Hủy một job (dù đang chạy hay đang chờ). |
| scancel \-u \[username\] | Hủy tất cả các job của bạn. |
| sinfo | Hiển thị thông tin về trạng thái của các node và partition trong cluster. |
| srun \[options\] \[command\] | Chạy một tác vụ tương tác (ví dụ: srun \--gpus=1 \--pty /bin/bash). |
| salloc \[options\] | Xin cấp phát tài nguyên (ví dụ: 1 node, 4 GPU) để sử dụng tương tác. |
| sacct \-j \[job\_id\] | Xem thông tin lịch sử chi tiết về một công việc đã hoàn thành hoặc thất bại. |

### **2\. Ví dụ Tùy chọn sbatch (trong file .slurm)**

| Tùy chọn | Mô tả |
| :---- | :---- |
| \#SBATCH \--output=slurm-%j.out | Chuyển hướng stdout vào tệp, %j là Job ID. |
| \#SBATCH \--error=slurm-%j.err | Chuyển hướng stderr vào tệp. |
| \#SBATCH \--gres=gpu:a100:2 | Yêu cầu 2 GPU loại A100. |
| \#SBATCH \--gres=fabric:ib:1 | Yêu cầu kết nối mạng tốc độ cao (InfiniBand). |
| \#SBATCH \--mail-type=ALL | Gửi email khi job bắt đầu, kết thúc, hoặc thất bại. |
| \#SBATCH \--mail-user=your@email.com | Địa chỉ email để nhận thông báo. |

## **🐳 Docker & Containers (Đóng gói)**

Đóng gói code và thư viện để đảm bảo code chạy ở mọi nơi.

### **1\. Lệnh Docker CLI**

| Lệnh | Mô tả |
| :---- | :---- |
| docker build \-t \[image\_name\] . | Xây dựng (build) một image từ Dockerfile trong thư mục hiện tại. |
| docker run \-it \--gpus all \[image\_name\] | Chạy một container từ image, cấp cho nó quyền truy cập tất cả GPU . |
| docker ps | Liệt kê các container đang chạy. |
| docker ps \-a | Liệt kê tất cả container (kể cả đã dừng). |
| docker images | Liệt kê tất cả các image đã build hoặc tải về. |
| docker exec \-it \[container\_id\] /bin/bash | Mở một terminal (bash) bên trong một container đang chạy (để debug). |
| docker pull \[image\_name\] | Tải về một image từ một registry (như Docker Hub). |
| docker stop \[container\_id\] | Dừng một container đang chạy. |
| docker rm \[container\_id\] | Xóa một container (đã dừng). |
| docker buildx build \--cache-from ... \--cache-to ... | Xây dựng với cache phân tán (cho CI/CD). |

### **2\. Dockerfile Tối ưu (BuildKit)**

| Cú pháp trong Dockerfile | Mô tả |
| :---- | :---- |
| RUN \--mount=type=cache,target=/root/.cache/pip ... | Sử dụng cache cho trình quản lý gói (pip, apt) để tăng tốc độ build. |
| RUN \--mount=type=bind,source=.,target=/src ... | Gắn kết (bind) mã nguồn vào quá trình build mà không sao chép. |
| FROM builder as build ... FROM base COPY \--from=build ... | Sử dụng multi-stage build để giữ image cuối cùng nhỏ gọn. |

## **☁️ Cloud CLI (AWS, Azure, GCP)**

Giao diện dòng lệnh để quản lý tài nguyên và gửi các công việc huấn luyện trên các nền tảng đám mây lớn.

### **1\. AWS (Amazon Web Services)**

| Lệnh | Mô tả |
| :---- | :---- |
| aws s3 sync ./local s3://bucket \--delete | Đồng bộ hóa một thư mục cục bộ lên S3, xóa các tệp không còn ở nguồn. |
| aws sagemaker create-training-job \--cli-input-json file://job.json | Gửi một công việc huấn luyện SageMaker bằng cách sử dụng tệp cấu hình JSON. |
| aws sagemaker list-training-jobs \--status-equals InProgress | Liệt kê các công việc huấn luyện đang chạy. |
| aws sagemaker describe-training-job \--training-job-name \[tên\] | Lấy thông tin chi tiết của một công việc huấn luyện cụ thể. |

### **2\. Azure**

| Lệnh | Mô tả |
| :---- | :---- |
| az storage blob upload-batch \-s ./data \-d mycontainer | Tải lên toàn bộ một thư mục (batch) lên Azure Blob Storage. |
| az storage blob download-batch \-s mycontainer \-d ./local\_data | Tải xuống toàn bộ một container về máy. |
| az ml job create \--file job.yaml \--stream | Gửi một công việc Azure ML từ tệp YAML và theo dõi log (stream) trực tiếp. |
| az ml job list | Liệt kê tất cả các công việc Azure ML. |
| az ml job show \--name \[tên\_job\] | Hiển thị chi tiết của một công việc cụ thể. |

### **3\. GCP (Google Cloud Platform)**

| Lệnh | Mô tả |
| :---- | :---- |
| gcloud storage rsync ./data gs://my-bucket/data | Đồng bộ hóa (rsync) một thư mục cục bộ lên Google Cloud Storage. |
| gcloud ai custom-jobs create \--local-package-path=./src ... | Tự động đóng gói và gửi một công việc huấn luyện Vertex AI từ mã nguồn cục bộ. |
| gcloud ai custom-jobs create \--config=config.yaml | Gửi một công việc huấn luyện Vertex AI bằng tệp cấu hình YAML. |
| gcloud ai custom-jobs list | Liệt kê các công việc tùy chỉnh trên Vertex AI. |
| gcloud ai custom-jobs describe \[job\_id\] | Lấy thông tin chi tiết của một công việc cụ thể. |

## **🧠 MLOps & AI Tools (MLflow, KFP, vLLM)**

Các công cụ chuyên dụng cho vòng đời AI/ML, theo dõi thử nghiệm và điều phối pipeline.

### **1\. AI/ML Chung**

| Lệnh | Mô tả |
| :---- | :---- |
| pip install \-r requirements.txt | Cài đặt tất cả các package được liệt kê trong file requirements.txt. |
| pip freeze \> requirements.txt | Ghi lại tất cả package và phiên bản của chúng vào file. |
| python \-m vllm.entrypoints.api\_server ... | (vLLM) Khởi chạy server API của vLLM để serving model. |
| ... \--model \[model\_name\] \--tensor-parallel-size \[N\] | (vLLM) Các tham số phổ biến: chỉ định model và số GPU để chạy song song. |
| wandb login | Đăng nhập vào Weights & Biases (dịch vụ theo dõi thí nghiệm). |
| tensorboard \--logdir=runs | Khởi chạy TensorBoard để xem log training (thường lưu trong thư mục runs). |

### **2\. MLflow**

| Lệnh | Mô tả |
| :---- | :---- |
| mlflow server \--backend-store-uri sqlite:///mlflow.db \--default-artifact-root ./mlruns | Khởi chạy một máy chủ MLflow theo dõi cục bộ. |
| mlflow experiments create \-n \[tên\_thử\_nghiệm\] | Tạo một thử nghiệm (experiment) mới. |
| mlflow run . \-P alpha=0.5 | Thực thi một dự án MLflow (từ file MLproject) với tham số. |
| mlflow models serve \-m "models:/MyModel/Production" \-p 5001 | Phục vụ (serve) một mô hình đã đăng ký từ Model Registry trên cổng 5001\. |

### **3\. Kubeflow Pipelines (KFP)**

| Lệnh | Mô tả |
| :---- | :---- |
| kfp dsl compile \--py my\_pipeline.py \--output pipeline.yaml | Biên dịch một pipeline KFP viết bằng Python sang tệp YAML. |
| kfp pipeline create \-p "My Pipeline" pipeline.yaml | Tải (upload) một pipeline đã biên dịch lên Kubeflow. |
| kfp pipeline create-version \-p \[pipeline\_id\] pipeline.yaml | Tải lên một phiên bản mới cho một pipeline đã tồn tại. |
| kfp run create \-e "My Experiment" \-f pipeline.yaml | Tạo và khởi chạy một lần chạy (run) pipeline từ tệp YAML. |
| kfp run list | Liệt kê tất cả các lần chạy pipeline. |
| kfp run get \[run\_id\] | Lấy thông tin chi tiết của một lần chạy cụ thể. |

## **🤗 Hugging Face CLI (hf)**

Công cụ để tương tác với Hugging Face Hub (thay thế cho huggingface-cli cũ).

| Lệnh | Mô tả |
| :---- | :---- |
| pip install huggingface\_hub | Cài đặt thư viện CLI. |
| hf auth login | Đăng nhập vào tài khoản Hugging Face của bạn (cần token). |
| hf auth whoami | Kiểm tra xem bạn đang đăng nhập bằng tài khoản nào. |
| hf repo create \[repo\_name\] | Tạo một repository mới trên Hub. |
| hf upload \[repo\_id\] \[local\_path\] \[repo\_path\] | Tải (upload) một file hoặc thư mục từ \[local\_path\] lên \[repo\_path\] trên Hub. |
| hf download \[repo\_id\] \[file\_trong\_repo\] | Tải (download) một file cụ thể từ repo về máy. |
| hf download \[repo\_id\] \--include "\*.safetensors" \--local-dir ./models | Tải (download) toàn bộ snapshot của một repository, chỉ bao gồm các tệp .safetensors. |
| git-lfs install | Cài đặt Git LFS (Large File Storage) để xử lý các file model lớn. |
| git-lfs track "\*.safetensors" | Yêu cầu Git LFS theo dõi tất cả các file có đuôi .safetensors. |

## **💿 DVC (Data Version Control)**

Quản lý phiên bản cho data, model, và các file lớn mà Git không thể xử lý.

| Lệnh | Mô tả |
| :---- | :---- |
| pip install dvc\[s3,gcs,local\] | Cài đặt DVC và các thư viện hỗ trợ cloud (s3, gcs...) hoặc local . |
| dvc init | Khởi tạo DVC trong một project (tương tự git init). |
| dvc remote add \-d s3\_remote s3://my-bucket/data | (Ví dụ: Cloud remote) Thêm S3 làm kho lưu trữ data. |
| dvc remote add \-d local\_remote /path/to/storage | (Ví dụ: Local remote) Thêm một thư mục trên máy làm kho lưu trữ. |
| dvc add \[tên\_file\_hoặc\_thư\_mục\] | Bắt đầu theo dõi một file/thư mục data (ví dụ: dvc add data/images). |
| dvc commit | Lưu lại thay đổi của file/thư mục đã được theo dõi vào cache (sau khi file gốc thay đổi). |
| dvc push \-r \[tên\_remote\] | Đẩy (upload) các file data (đã commit) lên kho lưu trữ từ xa (ví dụ: dvc push \-r local\_remote). |
| dvc pull \-r \[tên\_remote\] | Tải về (download) các file data từ kho lưu trữ từ xa (khi clone project về máy mới). |
| git checkout \[branch\] && dvc pull | Workflow chuẩn: Chuyển nhánh bằng Git, sau đó chạy dvc pull để đồng bộ data tương ứng. |

