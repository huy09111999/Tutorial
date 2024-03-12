- [1. Hướng dẫn dựng Moodle Web](#1-hướng-dẫn-dựng-moodle-web)
- [2. Hướng dẫn viết plugin kiểu Block của Moodle Web](#2-hướng-dẫn-viết-plugin-kiểu-block-của-moodle-web)
  - [2.1 Cấu trúc plugin kiểu Block](#21-cấu-trúc-plugin-kiểu-block)
  - [2.2 Tạo một plugin Block mới](#22-tạo-một-plugin-block-mới)
  - [2.3 Các API và phương thức có sẵn hỗ trợ](#23-các-api-và-phương-thức-có-sẵn-hỗ-trợ)
    - [Các Thuộc tính](#các-thuộc-tính)
    - [Các Phương thức](#các-phương-thức)

# 1. Hướng dẫn dựng Moodle Web
- Yêu cầu:
    - Ubuntu Desktop 20.04
    - Moodle 3.10.11
- Tham khảo cài đặt: https://geekrewind.com/how-to-install-moodle-with-apache2-and-lets-encrypt-on-ubuntu-18-04-16-04/

# 2. Hướng dẫn viết plugin kiểu Block của Moodle Web
## 2.1 Cấu trúc plugin kiểu Block
- Các plugin kiểu block được đặt trong thư mục **/blocks** của thư mục Gốc.
- Cấu trúc block_pluginname:
    ```
    blocks/pluginname/
    |-- db
    |   `-- access.php
    |-- lang
    |   `-- en
    |       `-- block_pluginname.php
    |-- pix
    |   `-- icon.png (optional)
    |-- block_pluginname.php
    |-- edit_form.php (optional)
    `-- version.php
    ```
- ### block_pluginname.php ###
    - Để định nghĩa những gì sẽ hiện thị trên màn hình.
    - Đường dẫn: **blocks/pluginname/block_pluginname.php**
    ```php
    <?php

    class block_pluginname extends block_base {

        /**
        * Khởi tạo khối.
        *
        * @return void
        */
        public function init() {
            $this->title = get_string('pluginname', 'block_pluginname');
        }

        /**
        * Lấy nội dung khối.
        *
        * @return string Khối HTML.
        */
        public function get_content() {
            global $OUTPUT;

            if ($this->content !== null) {
                return $this->content;
            }

            $this->content = new stdClass();
            $this->content->footer = '';

            // Thêm logic vào đây để xác định dữ liệu mẫu của bạn hoặc bất kỳ nội dung nào khác.
            $data = ['YOUR DATA GOES HERE'];

            $this->content->text = $OUTPUT->render_from_template('block_yourplugin/content', $data);

            return $this->content;
        }

        /**
        * Xác định những trang (page) nào khối này có thể được thêm vào.
        *
        * @return array các trang (page) nơi khối này có thể được thêm vào.
        */
        public function applicable_formats() {
            return [
                'admin' => false,
                'site-index' => true,
                'course-view' => true,
                'mod' => false,
                'my' => true,
            ];
        }
    }
    ```
- ### db/access.php ###
    - Chứa cấu hình các quy tắc kiểm soát truy cập của plugin. (Phân quyền xem, truy cập plugin).
    - Đường dẫn: **blocks/pluginname/db/access.php** 
    ```php
    <?php

    $capabilities = [
        'block/pluginname:myaddinstance' => [
            'captype' => 'write',
            'contextlevel' => CONTEXT_SYSTEM,
            'archetypes' => [
                'user' => CAP_ALLOW
            ],
            'clonepermissionsfrom' => 'moodle/my:manageblocks'
        ],
        'block/pluginname:addinstance' => [
            'riskbitmask' => RISK_SPAM | RISK_XSS,
            'captype' => 'write',
            'contextlevel' => CONTEXT_BLOCK,
            'archetypes' => [
                'editingteacher' => CAP_ALLOW,
                'manager' => CAP_ALLOW
            ],
            'clonepermissionsfrom' => 'moodle/site:manageblocks'
        ],
    ];
    ```

- ### lang/en/block_pluginname.php ###
    - Tập tin ngôn ngữ.
    - Mỗi plugin phải xác định một tập hợp các chuỗi ngôn ngữ có bản dịch tiếng Anh tối thiểu.
    - Đường dẫn tệp: **blocks/pluginname/lang/en/plugintype_pluginname.php**
    ```php
    <?php
    $string['pluginname'] = 'Pluginname block';
    $string['pluginname'] = 'Pluginname';
    $string['pluginname:addinstance'] = 'Add a new pluginname block';
    $string['pluginname:myaddinstance'] = 'Add a new pluginname block to the My Moodle page';
    ```

- ### version.php ###
    - Tệp này chứa siêu dữ liệu được sử dụng để mô tả plugin và bao gồm các thông tin như:
        - version: số phiên bản
        - dependencies: một danh sách các plugin phụ thuộc
        - requires: yêu cầu phiên bản Moodle tối thiểu
        - maturity: sự trưởng thành của plugin
    - Đường dẫn tệp: **blocks/pluginname/version.php**
    ```php
    <?php
        defined('MOODLE_INTERNAL') || die();

        $plugin->version = TODO;
        $plugin->requires = TODO;
        $plugin->supported = TODO;   // Available as of Moodle 3.9.0 or later.
        $plugin->incompatible = TODO;   // Available as of Moodle 3.9.0 or later.
        $plugin->component = 'block_pluginname';
        $plugin->maturity = MATURITY_STABLE;
        $plugin->release = 'TODO';

        $plugin->dependencies = [
            'mod_forum' => 2022042100,
            'mod_data' => 2022042100
        ];
    ```
- ### edit_form.php ###
    - Tệp này chỉ cần thiết nếu plugin của bạn có dạng cấu hình cụ thể. Nó không bắt buộc đối với hầu hết các plugin. 
    - Đường dẫn: **blocks/pluginname/edit_form.php**
    ```php
    <?php
        class block_pluginname_edit_form extends block_edit_form {
            protected function specific_definition($mform) {
            // Section header title according to language file.
            $mform->addElement('header', 'config_header', get_string('blocksettings', 'block'));
            // A sample string variable with a default value.
            $mform->addElement('text', 'config_text', get_string('blockstring', 'block_pluginname'));
            $mform->setDefault('config_text', 'default value');
            $mform->setType('config_text', PARAM_TEXT);
            }
        }
    ```
    - Chú ý: Tất cả tên trường của bạn cần bắt đầu bằng "config_" , nếu không chúng sẽ không được lưu và sẽ không có sẵn trong khối thông qua $this->config.

## 2.2 Tạo một plugin Block mới
- Cách dễ nhất để tạo plugin Block mới là sử dụng [Tool Pluginskel](https://moodle.org/plugins/tool_pluginskel). Bạn có thể sử dụng tệp yaml sau để tạo khung khối cơ bản.
    ```yaml
    ## This is an example recipe file that you can use as a template for your own plugins.
    ## See the list of all files it would generate:
    ##
    ##     php generate.php example.yaml --list-files
    ##
    ## View a particular file contents without actually writing it to the disk:
    ##
    ##     php generate.php example.yaml --file=version.php
    ##
    ## To see the full list of options, run:
    ##
    ##     php generate.php --help
    ##
    ---
    ## Frankenstyle component name.
    component: block_pluginname

    ## Human readable name of the plugin.
    name: Example block

    ## Human readable release number.
    release: "0.1.0"

    ## Plugin version number, e.g. 2016062100. Will be set to current date if left empty.
    #version: 2016121200

    ## Required Moodle version, e.g. 2015051100 or "2.9".
    requires: "3.11"

    ## Plugin maturity level. Possible options are MATURIY_ALPHA, MATURITY_BETA,
    ## MATURITY_RC or MATURIY_STABLE.
    maturity: MATURITY_BETA

    ## Copyright holder(s) of the generated files and classes.
    copyright: Year, You Name <your@email.address>

    ## Features flags can control generation of optional files/code fragments.
    features:
    readme: true
    license: true

    ## Privacy API implementation
    privacy:
    haspersonaldata: false
    uselegacypolyfill: false

    block_features:
    ## Creates the file edit_form.php
    edit_form: true

    ## Allows multiple instances of the block on the same course.
    instance_allow_multiple: false

    ## Choose where to display the block.
    applicable_formats:
        - page: all
        allowed: false
        - page: course-view
        allowed: true
        - page: course-view-social
        allowed: false

    ## Backup the block plugin.
    backup_moodle2:
        restore_task: true
        restore_stepslib: true
        backup_stepslib: true
        settingslib: true
        backup_elements:
        - name: elt
        restore_elements:
        - name: elt
            path: /path/to/file

    ## Capabilities defined by the plugin.
    capabilities:
    ## Required by block plugins.
    - name: myaddinstance
        title: Add a new pluginname block to the My dashboard
        captype: write
        contextlevel: CONTEXT_SYSTEM
        archetypes:
        - role: user
            permission: CAP_ALLOW
        clonepermissionsfrom: moodle/my:manageblocks

    - name: addinstance
        title: Add a new pluginname block
        captype: write
        contextlevel: CONTEXT_BLOCK
        archetypes:
        - role: editingteacher
            permission: CAP_ALLOW
        - role: manager
            permission: CAP_ALLOW
        clonepermissionsfrom: moodle/site:manageblocks

    ## Explicitly added strings
    lang_strings:
    - id: mycustomstring
        text: You can add 'extra' strings via the recipe file.
    - id: mycustomstring2
        text: Another string with {$a->some} placeholder.
    ```
## 2.3 Các API và phương thức có sẵn hỗ trợ
- Tất cả các khối phải cung cấp một lớp chính mở rộng lớp khối lõi. Tuy nhiên, có hai loại khối khác nhau:
    - `block_base`: Lớp cơ sở mặc định cho các khối nội dung.
    - `block_list`: Đối với các khối hiển thị một mục danh sách. 

- Tùy thuộc vào nhu cầu plugin của bạn, lớp chính của bạn trong **blocks/pluginname/block_pluginname.php** phải kế thừa **block_base** hoặc **block_list**.
- 
### Các Thuộc tính
- `$this->config`: Cấu hình phiên bản khối. Theo mặc định, nó là một đối tượng trống nhưng nếu khối có tệp edit_form.php thì nó sẽ là một đối tượng có dữ liệu biểu mẫu.
- `$this->content`: Biến này chứa tất cả nội dung thực tế được hiển thị bên trong mỗi khối. Các giá trị hợp lệ cho nó là NULL hoặc một đối tượng của lớp stdClass, phải có các biến thành viên cụ thể tùy thuộc vào lớp cơ sở khối mở rộng.
- `$this->page`: Đối tượng trang mà khối đang được hiển thị trên đó.
- `$this->context`: Đối tượng bối cảnh mà khối đang được hiển thị.
- `$this->title`: Tiêu đề của khối. 
### Các Phương thức
- `init()`: Phương thức init được gọi trước khi khối được hiển thị. Nó cần thiết cho tất cả các khối và mục đích của nó là cung cấp giá trị cho bất kỳ biến thành viên lớp nào cần khởi tạo. Tuy nhiên, nó được gọi trước khi $this->config được đặt, nếu plugin của bạn cần một số giá trị cấu hình để xác định các thuộc tính chung như tiêu đề khối, thì nó nên được thực hiện theo phương thức **specialization**.
- `specialization()`: Hàm này được gọi trên lớp con của bạn ngay sau khi một phiên bản được tải. Nó được sử dụng để tùy chỉnh tiêu đề và các thuộc tính khối khác tùy thuộc vào loại trang, ngữ cảnh, cấu hình,...
- `get_content(): string`: Để hiển thị nội dung mong muốn trên màn hình. Chú ý: hàm **get_content** có thể được gọi nhiều lần nên cần kiểm tra **$this->content** đã có dữ liệu chưa.
  ```php
    if ($this->content !== null) {
        return $this->content;
    }
  ```
- `applicable_formats(): array`: Khối sẽ có thể được hiển thị trên trang nào (ngữ cảnh). Mặc định khối sẽ được thêm vào tất cả các trang.
   
  - Mỗi trang trong Moodle có thể xác định tên loại trang riêng của nó. Tuy nhiên, có một số quy ước:

    - `['all' => true]`: khối có thể được sử dụng trong bất kỳ loại trang nào.
    - `site-index`: Trang đầu Moodle.
    - `course-view`: hiển thị trong Trang khóa học, độc lập với định dạng khóa học.
    - `course-view-FORMATNAME`: Thêm được khối ở định dạng khóa học "FORMATNAME". Ví dụ: `course-view-weeks` dành cho các khóa học có định dạng tuần.
    - `mod`: Bất kỳ trang hoạt động nào, độc lập với mô-đun.
    - `mod-MODNAME-view`: Thêm được khối ở module "MODNAME". Ví dụ: `mod-forum-view` dành cho diễn đàn.
    - `my`: Trang bảng điều khiển Moodle.
    - `admin`: Bất kỳ trang quản trị nào.
    ```php
    //ví dụ
    public function applicable_formats() {
        return [
            'admin' => false,
            'site-index' => false,
            'course-view' => true,
            'mod' => true,
            'my' => false
        ];
    }
    ```
- `instance_allow_multiple()`: mặc định: mỗi trang chỉ thêm được 1 phiên bản của 1 khối. Tuy nhiên, ta có thểm thêm nhiều phiên bản của 1 khối vào 1 trang.
    ```php
    public function instance_allow_multiple() {
        return true;
    }
    ```
- `hide_header(): bool`: ẩn tiêu đề khối.
    ```php
    public function hide_header() {
        return true;
    }
    ```
- `has_config(): bool`: thông báo cho Moodle Khối này có settings được viết trong tệp **/blocks/pluginname/settings.php**
    ```php
    function has_config() {
        return true;
    }
    ```
    ```php
    // Đọc config
    $settingvalue = get_config('block_simplehtml', 'settingname');
    ```