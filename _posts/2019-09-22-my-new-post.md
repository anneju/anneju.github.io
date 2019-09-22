---
layout: post
title: My New Post
date: 2019-09-22 22:35 +0800
---
### Action Mailer

1. 先 generate 內建的 Mailer，假如說我想生一系列關於 order的信，所以

   ```
   rails g mailer Order
   ```

2. 先看一下 rails 幫我們建了些什麼：

   ```
         create  app/mailers/order_mailer.rb                --> 類似 controller
         invoke  erb
         create    app/views/order_mailer                   --> 類似 view
         invoke  rspec
         create    spec/mailers/order_spec.rb
         create    spec/mailers/previews/order_preview.rb   --> 可以預覽
   ```

3. 先看看`app/mailers/order_mailer.rb`，可以發現裡面是空的，只是繼承 ApplicationMailer。

   打開 `app/mailers/application_mailer.rb`，裡面就是我們的預設值，預設信件都是從哪寄。

   ```ruby
   app/mailers/application_mailer.rb
   
   class ApplicationMailer < ActionMailer::Base
     default from: 'from@example.com'  <-- 預設寄件信箱
     layout 'mailer'
   end
   ```

   打開 app/view/layouts，裡面有 `mailer.html.erb` 和 `mailer.text.erb` ，前者是html設定檔，後面是文字。因為email裡的CSS常會被濾掉，所以裡面的CSS不要寫太複雜

4. 先設定方法：

   打開 app/mailers/order_mailer.rb ，可以在裡面寫方法：

   ```ruby
   class OrderMailer < ApplicationMailer
     def confirm_email
       mail to: 'test@gmail.com', subject: "訂單#{...}已付款" 
       ＃mail是個方法，後面接hash，所以順序不一定要這樣排
     end
   end
   ```

5. 接著去 orders_controller 去把要做這件事的地方加進去

   ```ruby
     def transaction
       if @order.may_pay?
         result = braintree_gateway.transaction....
         
         if result.success?
           @order.pay!
           # 寄 email
           OrderMailer.confirm_email.deliver_later
           redirect_to orders_path, notice: '刷卡已完成'
         else
           redirect_to orders_path, notice: '付款出現錯誤'
         end
       else
         redirect_to orders_path, notice: '訂單已完成並已付款'
       end
       
     end
   ```

   **註：可以看到 `OrderMailer.confirm_email`，可以發現是個類別方法。這是因為在 Action Mailer 定義的任何方法都會是類別方法，讓我們方便使用。**

   註：這邊的 .deliver_later 是為了避免像寄信、季報表這種比較花時間的是，先把它丟到背景工作區，晚一點再做。

6. 這時試著重新整理頁面，會看到log有出現 ActionView::MissingTemplate (Missing template order_mailer/confirm_email with "mailer"....這些字，表示還沒有view，所以要去做 view

7. 到 `app/views/order_mailer`這個資料夾，新增檔案 `confirm_email.html.erb`。可以這著在檔案裡先隨便打些字

   ```erb
   app/views/order_mailer/confirm_email.html.erb
   
   hi, 123
   ```

   接下來再次重整寄出e-mail的頁面，看一下log 檔，就可以看到剛剛打的字了。

   ```html
   <!DOCTYPE html>
   <html>
     <head>
       <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
       <style>
         /* Email styles need to be inline */
       </style>
     </head>
   
     <body>
       hi, 123
     </body>
   </html>
   ```

8. 接下來有兩種方法傳變數進去：

   8-1-1. 

   接下來把變數 order 帶進去

   ```ruby
   orders_controller  
   
   def transaction
       if @order.may_pay?
         result = braintree_gateway.transaction....
         
         if result.success?
           @order.pay!
           # 寄 email
           OrderMailer.confirm_email(@order).deliver_later
           redirect_to orders_path, notice: '刷卡已完成'
         else
           redirect_to orders_path, notice: '付款出現錯誤'
         end
       else
         redirect_to orders_path, notice: '訂單已完成並已付款'
       end
       
     end
   ```

   然後回去 mail controller 設定方法confirm_email

   ```ruby
   class OrderMailer < ApplicationMailer
     def confirm_email(order)
       mail to: order.user.email, subject: "訂單#{order.slug}已付款" 
       ＃mail是個方法，後面接hash，所以順序不一定要這樣排
     end
   end
   ```

   8-1-2. 

   也可以從 OrderMailer 這個 controller 傳實體變數進去 view 使用

   ```ruby
   class OrderMailer < ApplicationMailer
     def confirm_email(order)
       @name = 'xyz'
       mail to: order.user.email, subject: "訂單#{order.slug}已付款" 
       ＃mail是個方法，後面接hash，所以順序不一定要這樣排
     end
   end
   ```

   回去把 mailer view 帶入變數

   ```erb
   app/views/order_mailer/confirm_email.html.erb
   
   hi, <%= @name %>
   ```

   看log，剛剛帶入的實體變數已經出現了

   ```html
   <!DOCTYPE html>
   <html>
     <head>
       <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
       <style>
         /* Email styles need to be inline */
       </style>
     </head>
   
     <body>
       hi, xyz
     </body>
   </html>
   ```

   8-2-1. **使用 with**

   ​	把剛剛的 `OrderMailer.confirm_email(@order).deliver_later` 改成 `.with(order: @order)`

   ```ruby
   orders_controller  
   
   def transaction
       if @order.may_pay?
         result = braintree_gateway.transaction....
         
         if result.success?
           @order.pay!
           # 寄 email
           OrderMailer.with(order: @order).confirm_email.deliver_later
           redirect_to orders_path, notice: '刷卡已完成'
         else
           redirect_to orders_path, notice: '付款出現錯誤'
         end
       else
         redirect_to orders_path, notice: '訂單已完成並已付款'
       end
       
     end
   ```

   回到 mail controller，這邊可以用 `params[:order]` 把剛剛傳進來的 @order 捕捉下來

   ```ruby
   class OrderMailer < ApplicationMailer
     def confirm_email
       @order = params[:order]
       mail to: @order.user.email, subject: "訂單[#{@order.slug}]已付款"
     end
   end
   ```

   優點就是 confirm_email 不用帶參數進去，也不用去記傳入的參數的順序。

9. 預覽信件：

   到剛剛創建的預覽檔案 `spec/mailers/previews/order_preview.rb` ，上面有預覽網址：http://localhost:3000/rails/mailers，在裡面加入跟剛剛 OrderMailer 同名的方法（雖然不一定要同名，但同名比較好找），就可以看到連結。把剛剛在 OrdersController裡怎麼用的，就貼進去方法裡，把deliver_later刪掉，因為這只是要預覽。

   然後因為這邊是spec預覽，所以我們先利用 FactoryBot 做一個範例

   ```ruby
   # Preview all emails at http://localhost:3000/rails/mailers/order
   class OrderPreview < ActionMailer::Preview
     def confirm_email
       order = FactoryBot.create(:order)
       
       OrderMailer.with(order: @order).confirm_email
     end
   end
   ```

10. 開始寄信：

    參考Rails guide上關於Gmail設定這一段，在 config/environments/$RAILS_ENV.rb 設定（如果是開發環境就放在 development，如果是正式環境就是 production）

    ```
    config.action_mailer.delivery_method = :smtp  --> 用 smtp 的方式寄信
    config.action_mailer.smtp_settings = {        --> smtp 設定
      address:              'smtp.gmail.com',
      port:                 587,
      domain:               'example.com',
      user_name:            '<username>',
      password:             '<password>',
      authentication:       'plain',
      enable_starttls_auto: true }
    ```

    再回去寄信頁面重新整理，就可以順利寄信。