---
layout: post
---

<?php
/*
Plugin Name: 自建插件
Plugin URI: https://www.teagogo.cn
Description: 这是一个自建插件
Author: 洪向上
Version: 1.0
*/
// 将之前主题函数模板 functions.php 中的自定义代码加在此行下面就可以了
//
// 下面实现SMTP邮件发送功能，需要在wp-config.php中加入下面两行
// define('SMTP_USERNAME', 'mail@xxxx.com');//邮箱
// define('SMTP_PASSWORD', 'xxxxxxxx');//授权码，不是邮箱密码
//
add_action('phpmailer_init', 'mail_smtp');
function mail_smtp( $phpmailer ) {
  $phpmailer->FromName = '向上不懂茶'; // 发件人昵称
  $phpmailer->Host = 'smtp.qq.con'; // 邮箱SMTP服务器
  $phpmailer->Port = 465; // SMTP端口，不需要改
  $phpmailer->Username = SMTP_USERNAME;
  $phpmailer->Password = SMTP_PASSWORD;
  $phpmailer->From = 'teahacker@qq.com'; //邮箱地址
  $phpmailer->SMTPAuth = true;
  $phpmailer->SMTPSecure = 'ssl'; // 端口25时 留空，465时 ssl，不需要改
  $phpmailer->IsSMTP();
}

// 禁止新用户注册时发送邮件给管理员
add_filter( 'wp_new_user_notification_email_admin', '__return_false' );
// 禁止重置密码时发送邮件给管理员
add_filter( 'wp_password_change_notification_email', '__return_false' );
//
// 修改结算页面中的地址等字段，包括去掉邮政编码等
add_filter( 'woocommerce_default_address_fields', 'teagogo_remove_fields' );

function teagogo_remove_fields( $fields ) {

        unset( $fields[ 'address_2' ] );
        unset( $fields[ 'postcode' ] );
        unset( $fields[ 'company' ] );
        unset( $fields[ 'last_name' ] );
        return $fields;

}
// 修改结算页面名称
add_filter( 'woocommerce_checkout_fields' , 'custom_override_checkout_fields' );
function custom_override_checkout_fields( $fields ) {
     $fields['billing']['billing_first_name']['label'] = '收件人';
     $fields['billing']['billing_address_1']['label'] = '收货地址';
     $fields['billing']['billing_address_1']['placeholder'] = '请填写详细收件地址';
     $fields['billing']['billing_phone']['required'] = true;
     return $fields;
}
// 把电话改为必填
add_filter('woocommerce_checkout_fields', 'custom_override_checkout_fields');

function custom_override_checkout_fields($fields) {
    $fields['billing']['billing_phone']['required'] = true;
    return $fields;
}

