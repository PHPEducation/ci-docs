## Hướng dẫn sử dụng CI-v3 cho project Ede
**1) Config Civ3 cho**
- Trong folder root của project tạo file có tên `framgia-ci.yml` với nội dung như sau:
```
project_type: php
url: https://civ3-dev.s.vn
build:
  general_test:
    image: framgiaciteam/laravel-workspace:latest
    services:
      mysql_test:
        image: mysql:5.7
        environment:
          MYSQL_DATABASE: homestead_test
          MYSQL_USER: homestead_test
          MYSQL_PASSWORD: secret
          MYSQL_ROOT_PASSWORD: root
    prepare:
      - cp .env.civ3.example .env
      - composer install
      - yarn install # L
      - framgia-ci test-connect mysql_test 3306 60
      - php artisan migrate --database=mysql_test
test:
  phpcs:
    ignore: false
    command: echo '' | phpcs --standard=Framgia --report-checkstyle=.framgia-ci-reports/phpcs.xml app
  phpunit:
    ignore: false
    command:
      - php -dzend_extension=xdebug.so vendor/bin/phpunit
        --coverage-clover=.framgia-ci-reports/coverage-clover.xml
        --coverage-html=.framgia-ci-reports/coverage
cache:
  composer:
    folder: vendor
    file: composer.lock
```

- Tiếp đến trong folder root tạo tiếp file có tên `.env.civ3.example` có nội dung như sau:
```
APP_NAME=Laravel
APP_ENV=testing
APP_KEY=base64:tRodI3avqMgaquBm01DAOE0ZGLh1MtvFQAxxy49Q2fA=
APP_DEBUG=true
APP_LOG_LEVEL=debug
MIX_APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret

DB_TEST_CONNECTION=mysql_test
DB_TEST_HOST=mysql_test
DB_TEST_PORT=3306
DB_TEST_DATABASE=homestead_test
DB_TEST_USERNAME=homestead_test
DB_TEST_PASSWORD=secret

BROADCAST_DRIVER=log
CACHE_DRIVER=file
SESSION_DRIVER=file
SESSION_LIFETIME=120
QUEUE_DRIVER=sync

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1
```
- Mở file `config/database.php` trong phần
```
'connections' => [
    ...
]
```
Thêm nội dung sau vào:
```
'mysql_test' => [
    'driver' => 'mysql',
    'host' => env('DB_TEST_HOST', 'localhost'),
    'port' => env('DB_TEST_PORT', '3306'),
    'database' => env('DB_TEST_DATABASE', 'forge'),
    'username' => env('DB_TEST_USERNAME', 'forge'),
    'password' => env('DB_TEST_PASSWORD', ''),
    'charset' => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix' => '',
    'strict' => false,
    'engine' => null,
],
```
**2) Kiểm tra trạng thái pull request**
- Khi tạo pull request lên github, sẽ thấy trạng thái pull request đó đang được kiểm tra trên CI-v3
![available](https://raw.githubusercontent.com/dqhuy78/ci-docs/master/screenshot/1.png)
  - 1 - Pull request đang được chạy trên CI
  - 2 - Trạng thái hiện tại đang là running
  - 3 - Có thể click vào chữ detail để chuyển qua trang https://civ3-dev.framgia.vn để xem chi tiết việc kiểm tra pull request
- Sau khi pull request được kiểm tra hoàn tất sẽ trả về trạng thái như sau:
  - Pull reuqest succees (không có lỗi nào) - Có thể TO trainer để được merge
  ![available](https://raw.githubusercontent.com/dqhuy78/ci-docs/master/screenshot/2.png)
  - Trong trường hợp pull request có lỗi (có thể là lỗi convention, lỗi unit testm, ...) sẽ có trạng thái error như sau:
  ![available](https://raw.githubusercontent.com/dqhuy78/ci-docs/master/screenshot/3.png)
    - Nếu trainer đã add CIv3-Bot vào repostiory thì bot sẽ tự động comment vào các lỗi convention (nếu có) trong phần `Conversation` của pull request:
    ![available](https://raw.githubusercontent.com/dqhuy78/ci-docs/master/screenshot/4.png)
    (lỗi dùng tabs thay vì space)
    - Sau khi sửa hết lỗi và push lại CI sẽ kiểm tra lại và báo success như ảnh trước đó nếu không còn lỗi gì.
- Trong trường hợp bấm vào `detail` để chuyển qua trang xem chi tiết bản build của pull request sẽ như sau:
  ![available](https://raw.githubusercontent.com/dqhuy78/ci-docs/master/screenshot/5.png)
- Có thể click vào lệnh `framgia-ci run` để mở ra phần chi tiết chạy kiểm tra và lăn xuống cuối phần này để xem kết quả:
    ![available](https://raw.githubusercontent.com/dqhuy78/ci-docs/master/screenshot/6.png)
    (Ở đây kiểm tra phpcs bị lỗi dẫn đến việc kiểm trả pull request fail)
- Ở phần trên của trang có tab `Violations` có thể bấm vào để xem các lỗi convention mà pull request gặp phải:
    ![available](https://raw.githubusercontent.com/dqhuy78/ci-docs/master/screenshot/7.png)
    (ở đây có 3 lỗi convention php, có thể click vào chữ phpcs để nhảy đến danh sách các file bị lỗi))
    ![available](https://raw.githubusercontent.com/dqhuy78/ci-docs/master/screenshot/8.png)
    (click vào file lỗi sẽ hiện ra chi tiết lỗi của file đó)
    ![available](https://raw.githubusercontent.com/dqhuy78/ci-docs/master/screenshot/9.png)
