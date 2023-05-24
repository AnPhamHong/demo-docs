# BUILD 167. CR BANNER - http://redmine.ipicorp.co/issues/39182
#### Author: Phạm Hồng Ân - FE


## I/ Nhu cầu
- Thiết lập banner cho nhiều `Vùng, Chi nhánh, Đại lý` theo yêu cầu.

## II/ Giải pháp
### 1/ Update Tính năng __Quản lý banner trên ứng dụng__

#### 1.1/ Truy cập
- __Submenu__ Tin tức → chọn Quản lý Banner
- __Path__: ```/list/banner```

#### 1.2/ Danh sách banner
- __Title__: Quản lí banner trên ứng dụng
- __API__: ```/banner/get_list_banner_cms```
- __Params__

| STT | params | defined_key | note |
|:---:|:------:|:-----------:|:----:|
|1| key | | Tìm kiếm theo tiêu đề |
|2| agencyId| | multichoice, api: ```agency/get_all_agency_active```
|3| businessRegionId| | multichoice, api: ```business_region/get_all```
|4| status| ```2- tất cả, 1- hoạt động, 0- ngừng hoạt động```|Trạng thái hoạt động banner
|5| start_date| |
|6| end_date| |
|7| page| |

- __Thông tin columns__

|STT|Tên cột|key|type|note|
|:---------------:|:------------------:|:---------------:|:----------------:|:----------------:|
|1|STT|`stt`|int|
|2|Tiêu đề|`title`|string|
|3|Hình ảnh banner|`img_url + image`|string|
|4|Vùng|`lsBussinesName`|array|
|5|Chi nhánh|`lsAgencyName`|array|
|6|Đại lý|`lsUser`|array|DLC1 hoặc DLC2
|7|Trình chiếu trên|`banner_type`|string| S1, S2, S1:S2
|8|Ngày bắt đầu|`start_date`|int| #hhh#:#mm# #DD#/#MM#/#YYYY#
|9|Ngày kết thúc|`end_date`|int| #hhh#:#mm# #DD#/#MM#/#YYYY#
|10|Số giây tự động tắt|`countdown_time`|int|Thông tin ND
|11|Trạng thái|`status`|int| 1 - hoạt động, 0 - ngừng hoạt động

### 2/ Thêm đối tượng trình chiếu trên banner

#### 2.1/ Truy cập
- Tại page __Quản lí banner trên ứng dụng__ → click __Tạo mới__ → redirect đến page __Thêm đối tượng trình chiếu trên banner__
- __Path__: ```/banner/create/0```
- API sử dụng: ```banner/create_banner_cms```
```
    Request: 
     {
          "banner_type": 0, // loại trình chiếu: "S1" - 1, "S2" - 2, "S1:S2" - 0
          "conditionJoin": { // thông tin của Nhóm đối tượng áp dụng chương trình
                "conditionType": 0, // "Tạo điều kiện lọc" - 0, "Chọn theo Danh sách DLC1" - 1, "Chọn theo Danh sách DLC2" -2
                "lsAgencyId": [0], // ID chi nhánh theo vùng kinh doanh
                "lsBusinessRegion": [0], // Vùng kinh doanh
                "lsUserId": [0], // nhóm C1, C2 → ID của đại lý
                "user_lever": 0, // Cấp người dùng | đối tượng DLC1, DLC2
          },
          "countdown_time": 0, // số giây tự động tắt
          "end_date": 0, // ngày kết thúc
          "id": 0,
          "image": "string", 
          "start_date": 0, // ngày bắt đầu
          "status": 0, // trạng thái: "Hoạt động" - 1, "Ngừng hoạt động" - 0
          "title": "string" // tiêu đề banner
    }
```

#### 2.2/ Rules component Nhóm đối tượng áp dụng chương trình
Bao gồm 3 options (chọn 1)

|conditionType|Tên|
|-----------|---|
|0|Tạo điều kiện lọc|
|1|Chọn theo danh sách DLC1|
|2|Chọn theo danh sách DLC2|

```
        - Điều kiện tham gia 1: Vùng kinh doanh (multichoice)
            + api lấy data vùng kinh doanh: /business_region/get_all
            + key: lsBusinessRegion
        - Điều kiện tham gia 2: Chi nhánh (theo vùng kinh doanh)
            + api: agency/get_agency_by_list_bussiness_region
            + params: list_region, key_word
```

```
   2. Chọn theo Danh sách Đại lý C1
        - api lấy data vùng kinh doanh: /user_agency/get_all_for_create_event
        - params: page, city_id, agency_id, keySearch
```

```
   3. Chọn theo Danh sách Đại lý C2
        - api lấy data vùng kinh doanh: /supplier/get_all_for_create_event
        - params: page, city_id, agency_id, key
```

#### 2.3/ Lưu ý:
- Tất cả các field thông tin là bắt buộc
- Nếu conditionType = 0, thì các chi nhánh được chọn phải được kiểm tra trước khi post api, chi nhánh buộc phải nằm trong list vùng kinh doanh được chọn ở điều kiện tham gia 2
- Loại trình chiếu

|banner_type|Tên|
|-----------|---|
|0|S1:S2|
|1|S1|
|2|S2|


- Loại trình chiếu phải tương thích với Nhóm đối tượng áp dụng

|conditionType|banner_type|
|-----------|-------------|
|0|0,1,2|
|1|1|
|2|2|

### 3/ Update UI & API __Chi tiết trình chiếu Banner cho ứng dụng__
- Truy cập: tại danh sách banner → click vào dòng banner muốn xem
- path: `/banner/detail/id`
- __API__: `/banner/detail_banner_cms`
- __Params__: `id` - id của banner
- __Update field__:


|title|key|component|
|-----|----|----|
|Hình ảnh|`image`|Hình ảnh
|Tiêu đề|`title`|Thông tin Banner
|Trình chiếu trên|`banner_type`|Thông tin Banner
|Số giây tự động tắt|`countdown_time`|Thông tin Banner
|Trạng thái|`status`|Thông tin Banner
|Ngày hiệu lực|`start_date`|Thời gian áp dụng
|Ngày kết thúc|`end_date`|Thời gian áp dụng
|Vùng kinh doanh|`lsBussinesName`|Nhóm đối tượng áp dụng chương trình
|Chi nhánh|`lsAgencyName`|Nhóm đối tượng áp dụng chương trình
|Danh sách Đại lý Cấp 1/Cấp 2|`lsUser`|Nhóm đối tượng áp dụng chương trình


### 4/ Chỉnh sửa trình chiếu banner cho ứng dụng
![Chỉnh sửa banner!](build_167_edit_banner.PNG "Edit banner")
