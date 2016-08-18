# Parcelable Serializable trong Java (Android)

## I. Serializable trong Java:

### 1. Giới thiệu Serializable

Java cung cấp một cơ chế, được gọi là đối tuợng tuần tự (object serialization) nơi mà các đối tuợng đuợc biểu diễn như một chuỗi các bytes đuợc sắp xếp tuần tự bao gồm dữ liệu của đối tượng cũng như thông tin về kiểu của đối tượng và kiểu dữ liệu đựoc lưu trữ trong đối tượng. Sau khi một đối tuợng đã được ghi và lưu trữ xuống thành file (tệp tin), nó có thể đọc từ tập tin và tái tạo lại đối tượng từ tập tin đã lưu trữ theo đúng trình tự đã được nạp trong quá trình tuần tự hóa, đây là quá trình giải tuần tự hóa (deserialized).

Nói dễ hiểu hơn Serialization là quá trình chuyển các cấu trúc dữ liệu và các đối tượng thành một định dạng có thể lưu trữ được (vào file, in-memory buffer, hoặc truyền qua network), sau đó có thể phục hồi lại các cấu trúc dữ liệu và đối tượng như ban đầu, trên cùng hoặc khác môi trường.

Để 1 đối tượng được áp dụng cơ chế tuần tự hóa trong Java, đối tuợng đó phải được cài đặt interface java.io.Serialiable và cả trong các đối tượng con của đối tuợng này. Đây là một giao diện trống, không có method nào cần cài đặt.

* Thông thường thuật toán Serialization sẽ thực hiện các công việc sau:

 + Ghi xuống các siêu dữ liệu (metadata) về class (ví dụ như tên của class, version của class, tổng số các field của class,….) , của đối tượng đó.
 + Ghi đệ quy các thông tin chi tiết của các lớp cha cho tới khi nó gặp class Object. 
 + Sau khi hoàn tất việc ghi các siêu dữ liệu, tiến trình sẽ bắt đầu ghi các dữ liệu thật sự của các đối tượng

* Tuần tự hóa có ba mục đích chính sau

 + Cơ chế ổn định: Nếu luồng được sử dụng là FileOuputStream, thì dữ liệu sẽ được tự động ghi vào tệp.
 + Cơ chế sao chép: Nếu luồng được sử dụng là ByteArrayObjectOuput, thì dữ liệu sẽ được ghi vào một mảng byte trong bộ nhớ. Mảng byte này sau đó có thể được sử dụng để tạo ra các bản sao của các đối tượng ban đầu.
 + Nếu luồng đang được sử dụng xuất phát từ một Socket thì dữ liệu sẽ được tự động gửi đi tới Socket nhận, khi đó một chương trình khác sẽ quyết định phải làm gì đối với dữ liệu nhận được.

_Một điều quan trọng khác cần chú ý là việc sử dụng tuần tự hóa độc lập với thuật toán tuần tự hóa._

### 2. Tại sao lại cần đến Serialization?

* Một hệ thống enterprise điển hình thường có các thành phần nằm phân tán rải rác trên các hệ thống và mạng khác nhau. Trong Java mọi thứ đều được miêu tả như là một object. Nếu 2 thành phần Java cần liên lạc với nhau, ta cần phải có một cơ chế để chúng trao đổi dữ liệu. Serialization được định nghĩa cho mục đích này, và các thành phần Java sẽ sử dụng giao thức (protocol) này để truyền các object qua lại với nhau.
* Có thể dùng trao đối dữ liệu giữa 2 hệ thông khác nhau sử dụng thuật tóan tuần tự hóa mà không phụ thuộc vào nền tảng giữa chúng.

## II, Parcelable:

### 1. Giới thiệu

Theo các kỹ sư google, sử dụng đối tượng được cài đặt Parcelable sẽ chạy nhanh hơn đáng kể so với việc sử dụng Serializable. Một trong những lý do cho việc này là lớp này đuợc sử dụng rõ ràng về quá trình tuần tự thay vì sử dụng sự ánh xạ (reflection) để suy ra nó do đó mã đã được tối ưu hóa và tạo ra ít đối tuợng rác hơn cho mục đích này.

Tuy nhiên, việc thực hiện Parcelable không nhanh chóng và đơn giản như triển khai Serialization, phải triển khai một số lượng mã đáng kể để phục vụ cho điều này.

### 2. Triển khai

Parcelable thường được sử dụng để gửi dữ liệu (dạng Object) giữa các activity với nhau thông qua Bunble gửi cùng Intent. Để làm điều này 1 đối tượng phải được cài đặt giao diện Parcelable và ghi đè phương thức writeToParcel() trong lớp đó. Trong phương thức này triển khai ghi tất cả dữ liệu có trong lớp tới Parcel, tiếp theo triển khai một đối tuợng static final Parcelable.Creator để giải tuần tự tái tạo lại Java Object. Ví dụ:
```
 public class MyParcelable implements Parcelable {
     private int mData;

     public int describeContents() {
         return 0;
     }

     public void writeToParcel(Parcel out, int flags) {
         out.writeInt(mData);
     }

     public static final Parcelable.Creator<MyParcelable> CREATOR
             = new Parcelable.Creator<MyParcelable>() {
         public MyParcelable createFromParcel(Parcel in) {
             return new MyParcelable(in);
         }

         public MyParcelable[] newArray(int size) {
             return new MyParcelable[size];
         }
     };

     private MyParcelable(Parcel in) {
         mData = in.readInt();
     }
 }
```
Sau đó đưa đối tượng sau khi được đổ dữ liệu vào Bundle để truyền theo Intent tới Activity khác.
```
MyParcelable obj = new MyParcelable("Your Data to Constructor"); 
Bundle b = new Bundle();
b.putParcelable("ParcelKey", obj);
Intent i = new Intent(mContext, TargetActivity.class);
i.putExtras(b);
mContext.startActivity(i);
```
Để nhận về đối tượng vừa được gửi đi, bên Activity đích triển khai mã sau:
```
Bundle b = getIntent().getExtras();
MyParcelable obj = (MyParcelable) b. getParcelable("ParcelKey");
```

## III. Ưu nhược điểm

### Serializable:

* Ưu điểm:
  * Cực kì dễ triển khai, ít yêu cầu kèm theo
  * Có thể ghi đối tượng xuống dạng tệp tin để lưu trữ xuống ổ đĩa, và có thể đọc ở nhìu hệ thống khác nhau
* Nhược điểm:
  * Nó có tốc độ chậm, sinh ra nhiều đối tượng rác (garbage )
  * Nó rất khó để bảo trì (maintain) nếu bạn thay đổi cấu trúc của class.
### Parcelable:

* Ưu điểm:
  * Nó nhanh hơn Serializable
  * Dễ dàng đánh phiên bản cho đối tượng
  * Kiểm soát được dữ liệu tuần tự
* Nhược điểm:
  * Nó phụ thuộc vào nên tảng
  * Không thể lưu trữ xuống ổ đĩa (no disk cache)
  * Tốn nhiều mã nguồn để triển khai

## IV. Kết luận

* Hai giao diện đều có ưu và nhược điểm riêng, phụ thuộc vào bài toán được yêu cầu chúng ta sẽ sử dụng chúng 1 cách hợp lý.

* Giải pháp của tôi kết hợp Parcelable và Serializable độc đáo với nhau để bạn có được tốc độ của Parcelable khi truyền dữ liệu giữa các activity và sự tiện lợi của Serializable khi lưu vào đĩa (kèm một vài tối ưu để làm Serializable nhanh hơn) hoặc khi truyền dữ liệu thông qua các các giao thức mạng.
