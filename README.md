## แนวทางการทำงาน ESP32 project cook book
1. ระบุตัวอย่างที่ใช้ ว่าเอามาจากตัวอย่างไหน
  ใช้ https_server_simple
![image](https://github.com/user-attachments/assets/cf72827d-e588-419d-8ab3-c9a365efe9c9)

3. ระบุว่า จะแก้ไขตรงไหนบ้าง เพื่ออะไร
   ส่งค่าวัดแสงขึ้นไปบน web แก้ในส่วนของ main.c
```
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <esp_log.h>
#include <nvs_flash.h>
#include <sys/param.h>
#include "esp_netif.h"
#include "protocol_examples_common.h"
#include "esp_tls_crypto.h"
#include <esp_http_server.h>
#include "esp_event.h"
#include "esp_adc/adc_cali.h"
#include "esp_adc/adc_oneshot.h"

#define EXAMPLE_HTTP_QUERY_KEY_MAX_LEN  (64)
static adc_oneshot_unit_handle_t adc1_handle;
static const char *TAG = "example";

char analogtxt[128];
int adc_raw;

// ฟังก์ชันสำหรับการแสดงค่า LDR
static esp_err_t hello_get_handler(httpd_req_t *req)
{
    // อ่านค่าจาก LDR (ADC)
    adc_oneshot_read(adc1_handle, ADC_CHANNEL_0, &adc_raw);

    // อัพเดทข้อความที่จะแสดงผล
    sprintf(analogtxt, "<H1> LDR Value = %d </H1>", adc_raw);

    // ส่งข้อมูลไปยังผู้ใช้
    httpd_resp_send(req, analogtxt, HTTPD_RESP_USE_STRLEN);
    
    return ESP_OK;
}

static const httpd_uri_t hello = {
    .uri       = "/hello",
    .method    = HTTP_GET,
    .handler   = hello_get_handler,
    .user_ctx  = analogtxt
};

static httpd_handle_t start_webserver(void)
{
    httpd_handle_t server = NULL;
    httpd_config_t config = HTTPD_DEFAULT_CONFIG();

    ESP_LOGI(TAG, "Starting server on port: '%d'", config.server_port);
    if (httpd_start(&server, &config) == ESP_OK) {
        ESP_LOGI(TAG, "Registering URI handlers");
        httpd_register_uri_handler(server, &hello);
        return server;
    }

    ESP_LOGI(TAG, "Error starting server!");
    return NULL;
}

void app_main(void)
{
    static httpd_handle_t server = NULL;

    ESP_ERROR_CHECK(nvs_flash_init());
    ESP_ERROR_CHECK(esp_netif_init());
    ESP_ERROR_CHECK(esp_event_loop_create_default());

    // ตั้งค่า ADC สำหรับ LDR
    adc_oneshot_unit_init_cfg_t init_config1 = {
        .unit_id = ADC_UNIT_1,
    };
    ESP_ERROR_CHECK(adc_oneshot_new_unit(&init_config1, &adc1_handle));

    adc_oneshot_chan_cfg_t config = {
        .bitwidth = ADC_BITWIDTH_DEFAULT,
        .atten = ADC_ATTEN_DB_11,
    };
    ESP_ERROR_CHECK(adc_oneshot_config_channel(adc1_handle, ADC_CHANNEL_0, &config));

    // เชื่อมต่อ Wi-Fi และเริ่ม HTTP server
    ESP_ERROR_CHECK(example_connect());
    server = start_webserver();

    while (server) {
        sleep(5);
    }
}

```
3. แสดงขั้นตอนการทำ project
 - สร้างexample และทำการ build ทดสอบก่อน
 - หลังจากนั้นทำการ เปลี่ยน config WiFi เป็น internet / hotsport ของตัวเอง
 - แก้ไขโค้ดในส่วนของ main.c ว่าต้องการให้หน้า web แสดงข้อความใดที่จะส่งข้อมูลจาก esp32 ขึ้นไป
 - ในส่วนนี้เลือก LDR วัดแสง และทำการเปลี่ยน โค้ดในส่วนของการต้องการให้วัดแสง (ข้อ2)
 - ทำการ build flash monitor
 - นำ ip ไปพิมพ์ใน บราวเซอร์
 -  ![image](https://github.com/user-attachments/assets/f2a25636-515c-4792-b83d-a5f624ab8a61)
5. แสดงผลการทำ project
- จะแสดงผลดังรูป![image](https://github.com/user-attachments/assets/17c99cf3-8a82-4d06-8cfc-6b3f237b47ee)
6. สรุปผลการทำ project 
- เป็นการสร้าง webserver ในการเรียกดูค่าวัดแสง LDR บนบอร์ด ESP32
