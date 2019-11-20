
## Sử dụng Nginx để config load balancing cho web server

Có một bài toán rất phổ biến trong thực tế là khi hệ thống của bạn có quá nhiều người sử dụng, lượng request mỗi giây, mỗi phút vô cùng lớn gọi về server của bạn. Lúc này, nếu chỉ có một server đơn lẻ, thì hệ thống của bạn không thể đáp ứng được cho tất cả các người dùng cùng một lúc. Giải pháp trước tiên, bạn nghĩ tới là sẽ config thêm nhiều con server khác nữa, để chia sẻ gánh nặng với một server ban đầu. Tuy nhiên, mỗi request của người dùng gửi về, thì bạn cũng không thể controll được việc server nào của bạn sẽ trả về dữ liệu cho request đó, khi đó dễ dẫn tới tình trạng một server hoạt động tối đa công suất, còn có những server chỉ hoạt động được 10, hay 20% công lực của nó => Dẫn tới việc, bài toán ban đầu không được cải thiện bao nhiêu, mà chi phí để duy trì server của bạn lúc này lại bị đội lên vài lần, tùy thuộc vào số server bạn đang có. Để giải quyết tất cả vấn đề trên, thì bộ cân bằng tải được ra đời. Bộ cân bằng (**Load balancer**) tải sẽ quyết định việc tải dữ liệu từ server nào để trả lại cho người dùng khi có request tới.

## 1. Load balancer là gì?

Ngày nay, thuật ngữ Load balancer không còn quá xa lạ với lập trình viên chúng ta nữa, bạn có thể dễ dàng tìm kiếm trên google với từ khóa load balacer là gì? bạn sẽ thu được hàng ngàn kết quả trả về trong vài phần trăm của giây, có thể tổng hợp ngắn gọn xúc tích lại định nghĩa load balacer lại như sau:

```
Load balancer là việc phân phối hiệu quả lưu lượng truy cập đến trên một nhóm backend server
```

## 2. Load balancer hoạt động như thế nào?

Hiểu một cách đơn giản, load balacer hoạt động tương tự như một "cảnh sát giao thông" (traffic cop) ở phía trước server và điều hướng (routing) các request của người dùng trên tất cả các server có khả năng thực hiện request đó, sao cho tối ưu về tốc độ và hiệu quả cao nhất, và đảm bảo là không có server nào hoạt động quá 100% công lực và cũng không có server nào hoạt động 10 hay 20 % công lực. Khi có một server làm việc nặng nhọc quá, lăn ra chết, thì load balance sẽ tự động điều hướng request tới những server khỏe hơn, đang chạy tốt. Hay khi có một server được thêm vào, load balancer sẽ tự động giao việc cho server mới đó, để nó hoạt động.

Chung quy lại, chức năng chính của load balacer như sau:

- phân phối các request một cách hiệu quả trên nhiều server

- đảm bảo tính khả dụng và độ tin cậy cao bằng cách chỉ gửi request tới những máy chủ đang hoạt động

- thêm vào hoặc bớt server một cách linh hoạt

Trên đây là một số thông tin tối thiểu cần biết về load balancer, các bạn có thể độc thêm thông tin về load balacer trên gg nhé.

## 3. Nginx là gì?

Nếu bạn là một kỹ sư lập trình web, chắc hẳn các bạn cũng đã biết tới apache và nginx là phần mêm web server rồi đúng k?

Tóm lại như sau: Nginx là web server có thể hoạt động như là email proxy, reverse proxy và load balancer. Cấu trúc của nó là bất đồng bộ và hướng sự kiện, vì vậy cho phép nó xử lý nhiều truy vấn cùng lúc. Nginx dễ dàng để mở rộng cho website hơn, đồng nghĩa với việc nó có thể đi theo suốt quá trình phát triển của website

## 4. Cài đặt nginx

Đầu tiên, cần setting một máy chủ mới, cái mà hoạt động như là load balancer của bạn, Bạn có thể lựa chọn một trong các hệ điều hành sau: Centos, Debian hay là Ubuntu, ... bất cứ cái nào mà bạn thích. Sau khi có host rồi, hãy cài đặt nginx vào host đó theo một trong các cách sau:

```
# Debian and Ubuntu
sudo apt-get update
# Then install the Nginx Open Source edition
sudo apt-get install nginx
```

```
# CentOS
# Install the extra packages repository
sudo yum install epel-release
# Update the repositories and install Nginx
sudo yum update
sudo yum install nginx
```

Sau khi cài đặt nginx thành công, hay vào thư mục cấu hình chính của nginx

```
cd /etc/nginx/
```

Tùy vào hệ điều hành máy chủ của bạn, mà các tệp cấu hình máy chủ web sẽ ở một trong hai nơi. Với Ubuntu hoặc DEbian thì file cấu hình ở trong /etc/nginx/sites-available/, bạn có thể dùng câu lệnh dưới đây để enable bất kỳ file cấu hình máy chủ mới nào

```
sudo ln -s /etc/nginx/sites-available/vhost /etc/nginx/sites-enabled/vhost
```

Đối với Centos, bạn có thể tìm kiếm file config ở đường dẫn  /etc/nginx/conf.d/

Sau khi cài đặt và config hoàn tất, hãy thử test lại xem việc cài đặt của bạn có gặp bất kỳ lỗi gì hay không nhé.

## 5. Config nginx như là một load balancer

Sau khi bạn đã cài đặt nginx và test để đảm bảo nginx đang chạy đúng trên máy chủ của bạn thì hãy tiến hành cài đặt nginx như là một load balance. Trước hết, mở tạo file load-balancer.conf với câu lệnh

```
sudo nano /etc/nginx/conf.d/load-balancer.conf
```

trong file này bạn sẽ định nghĩa 2 segment là upstream và server, xem ví dụ dưới đây

```
# Define which servers to include in the load balancing scheme. 
# It's best to use the servers' private IPs for better performance and security.
# You can find the private IPs at your UpCloud control panel Network section.
http {
   upstream backend {
      server 10.1.0.101; 
      server 10.1.0.102;
      server 10.1.0.103;
   }

   # This server accepts all traffic to port 80 and passes it to the upstream. 
   # Notice that the upstream name and the proxy_pass need to match.

   server {
      listen 80; 

      location / {
          proxy_pass http://backend;
      }
   }
}
```

Save file này lại rồi quit ra khỏi editor. Tiếp theo, disable file default config, cái mà bạn đã dùng nó để test sau khi cài đặt nginx thành công ở bước trên bằng cách

```
sudo rm /etc/nginx/sites-enabled/default
```

Đây là câu lệnh dùng cho Ubuntu và Deabin, còn với centos thì dùng lệnh

```
sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.disabled
```

Tiếp theo là restart lại nginx:

```
sudo systemctl restart nginx
```

Kiểm tra xem nginx có restart thành công hay không, trong trường hợp restart lỗi, hãy thử kiểm tra lại xem file /etc/nginx/conf.d/load-balancer.conf bạn vừa tạo để kiểm tra lại xem có lỗi đánh máy hay thiếu dấu , nào hay không. Khi bạn truy cập vào load balancer public IP trên trình duyệt của mình, bạn nên chuyển tới một trong các máy chủ back-end của mình

## 6. Load balancing methods

Cân bằng tải với nginx sử dụng thuật toán round-robin làm mặc định nếu không có phương pháp nào được khai báo, như trong ví dụ trên, Với thuật toán round-robin (quay vòng) mỗi máy chủ được chọn lần lượt theo thứ tự bạn đặt chúng trong file load-balancer.conf. Thuật toán này sẽ cân bằng số lượng request như nhau cho những request cần ít thời gian xử lý.

Một thuật toán khác là kết nối tối thiểu (least connections). Đây cũng là một phương pháp cơ bản, như tên cho thấy, phương pháp này điều hướng những request tới các máy chủ có ít kết nối nhất tại thời điểm đó. so với thuật toán quay vòng, nó hoạt động hiệu quả với với những ứng dụng mà mỗi request sẽ mất nhiều thơi gian xử lý để hoàn thành.

Để xử dụng được phương thức kết nối tối thiểu, hãy thêm tham số less_conn như ví dụ dưới đây

```
upstream backend {
   least_conn;
   server 10.1.0.101; 
   server 10.1.0.102;
   server 10.1.0.103;
}
```

Hai thuật toán round-robin và least connections đều tốt và thể hiện được vai trò của nó trong những trường hợp nhất định, tuy nhiện hạn chế của nó là không thể duy trì session trong mỗi phiên làm việc. Nếu ứng dụng của bạn yêu cầu người dùng đăng nhập sau đó chuyển tới một máy chủ back-end khác giống với máy chủ mà họ đã đăng nhập trước đó, thì hai phương thức trên đều không thẻ xử dụng được, lúc này bạn phải xử dụng phương pháp IP hashing (băm IP),là phương thức sử dụng địa chỉ ip của người dùng làm khóa để xác minh máy chủ nào sẽ được chọn để phục vụ yêu cầu. Điều này cho phép mỗi lần người dùng truy cập đều được hướng tới một máy chủ, với điều kiện là máy chủ đó khả dụng và ip của khách chưa thay đổi.

Để sử dụng phương thức ip hashing thì thêm params ip_hash như sau:

```
upstream backend {
   ip_hash;
   server 10.1.0.101; 
   server 10.1.0.102;
   server 10.1.0.103;
}
```

Trong việc cài đặt máy chủ, tất nhiên sẽ sinh ra trường hợp là các máy chủ khác nhau thì tài nguyên trên từng máy sẽ không giống nhau, chúng ta có thể mong muốn một số máy chủ làm việc nhiều hơn một số máy còn lại. Xác định trọng số cho máy chủ cho phép bạn tinh chỉnh thêm load balancer với nginx. Với phương pháp này, máy chủ có trọng số cao nhất sẽ được chọn thường xuyên nhất. Hiểu đơn giản hơn thì là ông nào khỏe nhất thì phải làm nhiều nhất. =)). Phương pháp này nghe có vẻ không được fair. 

Để setting được điều này thì bạn làm như sau:

```
upstream backend {
   server 10.1.0.101 weight=4; 
   server 10.1.0.102 weight=2;
   server 10.1.0.103;
}
```

Với ví dụ trên, bạn có thể thấy máy chủ được khai báo đầu tiên có trọng số cao nhất là 4, vì thế nó sẽ thường xuyên được gọi để xử lý các yêu cầu. 

## 7. cân bằng tải với HTTPS

Khi bạn enable HTTPS cho website của mình, nó là cách tốt nhất để bảo vệ nguời dùng và dữ liệu của họ khi họ truy cập vào website của bạn.

Để sử dụng load balancer với website https thì vô cùng đơn giản. Tất cả những gì bạn cần là thêm một section server khác vào file load balancer config, cái mà lắng nghe https trafic tại cổng 443 với SSL. Xem ví dụ dưới đây

Mở file config load-balncer.conf của bạn 

```
sudo nano /etc/nginx/conf.d/load-balancer.conf
```

Thêm một server mới vào như sau:

```
server {
   listen 443 ssl;
   server_name domain_name;
   ssl_certificate /etc/letsencrypt/live/domain_name/cert.pem;
   ssl_certificate_key /etc/letsencrypt/live/domain_name/privkey.pem;
   ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

   location / {
      proxy_pass http://backend;
   }
}
```

save file lại và restart với lệnh sau:

```
sudo systemctl restart nginx
```

## 8. Kiểm tra

Đẻ biết máy chủ nào khả dụng, nginx cho phép chúng ta kiểm tra tình trạng máy chủ bằng reverse proxy. Neus một máy chủ không trả lời yêu cầu hoặc trả lời lỗi, nginx sẽ note là máy chủ đã bị lỗi. Nó sẽ tránh chuyển các request tới máy chủ đó trong một khoảng thời gian

Số lần thử kết nối không thành công liên tiếp trong một khoảng thời gian có thể đượng định nghĩa trong file config. Bởi tham số max_fail cho mỗi máy chủ. Nếu không có tham số này thì mặc định là 1. Nếu bạn đặt max_fail là 0 tức là bạn sẽ disable chức năng helth check của nginx

Nếu max_fail mà lớn hơn 1, thì lần thất bại tiếp theo phải xảy ra trong một khung thời gian cụ thể để không đếm được. Khoảng thời gian này được chỉ định bởi tham số fail_timeout. Mặc định của gía trị này là 10 giây.

Sau khi máy chủ bị note là không thành công và thời gian bời fail_timeout đã trôi qua, nginx sẽ thăm do máy chủ đó với các yêu cầu của máy khách. Nếu dữ liệu được trả lại thành công, máy chủ lại được đánh dấu là hoạt động và nằm trong cân bằng tải như bình thường

Xem ví dụ dưới đây

```
upstream backend {
   server 10.1.0.101 weight=5;
   server 10.1.0.102 max_fails=3 fail_timeout=30s;
   server 10.1.0.103;
}
```

Trên đây là một vài khái niệm cơ bản về load balancer, hy vọng sẽ giúp ích mọi nguời trong thực tế. Ch
