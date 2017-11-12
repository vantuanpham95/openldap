# TÌM HIỂU VỀ OPENLDAP - BUILD OPENLDAP REPLICATION 
## I. LÝ THUYẾT 
### 1. Tổng quan LDAP - Lightweight Directory Access Protocol 
LDAP là chuẩn cho dịch vụ thư mục (directory access) phát triển dựa trên X500 và chạy trên OSI. LDAP chỉ là giao thức, không hỗ trợ xử lý như cơ sở dữ liệu mà nó cần một nơi lưu trữ backend và xử lý dữ liệu tại đó. 

LDAP là giao thức truy cập dạng client-server, mô hình dạng cây. 

### 2. Phương thức hoạt động của LDAP 
Một hay nhiều LDAP server chứa thông tin về cây thư mục (Directory Information Tree - DIT) Client kết nối đến server và gửi yêu cầu, server nếu có thông tin thì sẽ trả lời còn nếu không sẽ trỏ tới LDAP server khác để client lấy thông tin. 

Quá trình kết nối diễn ra như sau 

 * Connect: Client gửi một yêu cầu kết nối tới LDAP 
 * Bind: Client gửi thông tin xác thực 
 * Search: Client gửi yêu cầu tìm kiếm 
 * Interpret Search: server thực hiện xử lý tìm kiếm 
 * Results: server trả lại kết quả cho client 
 * Unbind: Client gửi yêu cầu đóng kết nối tới server 
 * Close connection: Server đóng kết nối, kết thúc quá trình 
### 3. Database của LDAP 
slapd là một LDAP directory server, chạy trên nhiều nền tảng khác nhau. Các tính năng như: 
 
 * LDAPv3: slđap hỗ trợ LDAP IPv4, IPv6 và Unix PC 
 * Simple Authentication and Security Layer: slapd hỗ trợ chứng thực và bảo mật dữ liệu dịch vụ SASL 
 * Transport Layer Security: slapd hỗ trợ sử dụng TLS và SSL 
slapd sử dụng 1 trong 2 database để lưu trữ dữ liệu hiện tại là hdb và bdb. BDB sử dụng Oracle Berkeley DB để lưu trữ dữ liệu, còn HDB cũng có tương tự như BDB nhưng hỗ trợ cơ sở dữ liệu dạng cây. Thường mặc định sử dụng HDB 
### 4. Lưu trữ thông tin của LDAP 
Ldif (LDAP Data Interchange Format) là một chuẩn định dạng file text chứa thông tin cấu hình LDAP và nội dung thư mục. File Ldif miêu tả các thuộc tính của entries mới để add thêm vào, hay dùng để sửa, xóa các entries đã tồn tại. Dữ liệu trong file LDIF phải tuân theo quy luật của LDAP theo schema. 
**Cú pháp của LDIF**

 * Có thể có 1 hoặc nhiều entries, mỗi tập entry khác nhau được phân cách nhau bởi 1 dòng trống 
 * "tên thuộc tính: giá trị" 
 * Một tập chỉ dẫn cú pháp làm sao để xử lý thông tin 
**Chú ý:** Các yêu cầu khi khai báo trong ldif 

 * Có thể chú thích bằng cách thêm # ở đầu dòng 
 * Thuộc tính được liệt kê bên trái dấu ":" và giá trị được biểu diễn bên phải 
 * Thuộc tính dn (distinguished name) phải là duy nhất 
**Ví dụ:** 2 file Ldif sau được sử dụng để thực hiện hành động với một entry trong directory cho một máy in. 

Description 
```
dn: cn=LaserPrinter1, ou=Devices, dc=vccloud, dc=vn 
objectclass: top 
objectclass: printer 
objectclass: epsonPrinter 
cn: LaserPrinter1 
resolution: 600 
description: In Room T17-VCCLOUD 
``` 

Modification 
```
dn: cn=LaserPrinter1, ou=Devices, dc=vccloud, dc=vn 
changetype: modify 
add: pagesPerMinute 
pagesPerminute: 6 
```
Dòng đầu tiên trong mỗi file LDIF là tên của entry, được gọi là dn - distinguished name chỉ ra chính xác entry cần làm việc. Mỗi entry được xác định bởi dn duy nhất 

File đầu tiên mô tả các thuộc tính của máy in để thêm vào trong cơ sở dữ liệu, file thứ 2 được sử dụng như là một input để câu lệnh ldapmodify thực hiện thay đổi, ở đây là thêm thuộc tính tốc độ in (pagesperminute) cho máy in đã được định nghĩa và thêm vào từ file đầu tiên.  

Các thuộc tính cơ bản trong file ldif: 

* dn: distinguished name: tên gọi phân biệt 
 * c: country: 2 ký tự viết tắt tên nước 
 * o: organization: tên tổ chức 
 * ou: organization unit: bộ phận của tổ chức 
 * objectclass: Giống như một khuôn mẫu của các dữ liệu cho các entry  
 * givenname: tên 
 * uid: id người dùng 
 * cn: common name: tên thường gọi 
 * telephonenumber: số điện thoại 
 * sn: surname: họ 
 * userPassword: mật khẩu 
 * mail: địa chỉ mail 
 * createTimestamp: thời gian khởi tạo entry 
 * creatorsName: tên người khởi tạo entry 
 * pwdchangedtime: thời gian đổi mật khẩu 
 * entryUUID: id của entry 
 ### 5. LDAP information model 
 LDAP information model mô tả cách xây dựng ra khối dữ liệu mà chúng ta có thể sử dụng để tạo ra thư mục, nó định nghĩa ra các kiểu dữ liệu và các thành phần thông tin cơ bản có thể chứa trong thư mục. 
 
 ![1]()
 
 
**Đặt tên trong LDAP:**

LDAP naming chính là cách để tham chiếu đến dữ liệu của mình. Đồng thời đây cũng chính là cách để chúng ta định nghĩa, sắp xếp và thay đổi các entry vào các thư mục để quản lý một cách hợp lý và khoa học. 

Tên của một entry trong LDAP được hình thành bằng cách nối tất cả các tên của từng entry cấp trên (cha) cho đến entry trên cùng (root), ví dụ tên của JohnDoe trong hình trên là JohnDoe.Developing.Organization.US.RootDSE 

Thành phần đầu tiên của một DN được gọi là RDN (relative distinguished name) RDN có thể giống nhau miễn DN khác nhau là được (các RDN giống nhau không thể có cùng cha) 

![2]()

**Aliases**

Các entry Aliases cho phép một entry trỏ đến một entry khác 

Cách dùng: thêm thuộc tính aliasedObjectName vào 1 entry và giá trị của thuộc tính đó là DN của entry mà ta muốn aliase entry này trỏ tới 

![3]()

--> Nếu có quá nhiều aliases entry sẽ tốn thêm nhiều chi phí cho việc tìm kiếm, vì nó phải tìm trên một môi trường rộng lớn hơn nhiều thậm chí có thể loops 

### 6. LDAP user authentication 
Tât cả các công việc tìm kiếm, thay đổi và thiết lập đối với directory server đều được kiểm soát bởi mức quyền của user được xác thực 

**Anonymous authentication**

Đăng nhập vào directory server với tên đăng nhập và mật khẩu là rỗng 

**Simple Authentication**

Đăng nhập vào directory server với tên đăng nhập và mật khẩu dưới dạng clear text 

Nếu mật khẩu đã qua một bước hash trong cơ sở dữ liệu thì server sẽ hash mật khẩu cleartext nhập theo hàm hash tương ứng và đối chiếu lại, nếu  trùng thì đăng nhập thành công 
**Authentication via SSL/TLS**

LDAP sẽ mã hóa trước khi thực hiện bất cứ hoạt động kết nối nào 

### 7. LDAP command lines 

**Ldapbind**

ldapbind được sử dụng để xác thực tới một directory server 

Cú pháp: ldapbind [options] 

Ví dụ: 

    ldapbind -h myhost -p 389 -D "cn=vccloudadmin" -w welcome 

--> ldapbind xác thực user có common name là vccloudadmin tới directory server myhost, cổng TCP 389 và sử dụng password là welcome 

**Ldapsearch**

ldapsearch được sử dụng để tìm kiếm một hoặc nhiều entries nhất định trong một directory, ldapserch mở một kết nối tới directory server, xác thực một người dùng để thực thi các truy vấn, tìm kiếm một hay nhiều entries dựa trên quyền của user và cuối cùng là in kết quả ra màn hình theo format được gõ trong lệnh 

Cú pháp: ldapsearch [options] filter [attributes] 

Ví dụ: 
```
ldapsearch -h myhost -p 389 -s base -b "ou=people, dc=acme, dc=com" \ 
"objectclass=*" 
```
--> Tìm kiếm trên directory myhost, cổng 389, phạm vi tìm kiếm (search scope -s) là base và phần tìm kiếm trong directory của base -b là DN: ou=people, dc=acme, dc=com. Bộ lọc tìm kiếm objectclass=* sẽ trả về tất cả các entries có giá trị objectclass bất kỳ. Không có một thuộc tính nào được trả về vì trong câu lệnh không yêu cầu. Câu lệnh này cũng là một câu lệnh tìm kiếm anonymous bởi vì tùy chọn authentications không được nhắc đến. Câu lệnh cũng có thể xuống dòng bằng cách sử dụng ký tự báo xuống dòng \ 

**Chú ý:** các phạm vi tìm kiếm 

 * Base: tìm kiếm ngay tại đối tượng (-s base -b "ou=people, dc=acme, dc=com")
 * Onelevel: tìm kiếm ngay dưới đối tượng (-s one -b "ou=people, dc=acme, dc=com") 
 * subtree: tìm kiếm toàn bộ subtree theo đối tượng (-s sub -b "ou=people, dc=acme, dc=com") 
 
 ![4]()
 
**Ngoai ra còn có các tham số:**

* derefAliases: Cho server biết rằng liệu aliases có bị bỏ qua hay không khi thực hiện tìm kiếm, có 4 giá trị là: 

  * neverderefaliases: thực hiện tìm kiếm không  bỏ qua aliases 

  * dereflnsearching: Bỏ qua các aliases trong các entries cấp dưới của base DN  và không quan tâm đến thuộc tính của Base DN 

  * DerefFindingBaseObject: tìm kiếm bỏ qua các aliases của base DN 
  
  * DerefAlways: Luôn bỏ qua aliases 
 
 * Cho server biết có tối đa bao nhiêu entries kết quả được trả về 

 * Quy định thời gian tối đa của việc thực hiện tìm kiếm 

 * attrOnly: true or false. Nếu đặt là true thì server chỉ gửi các kiểu thuộc tính của entry cho client mà không gửi giá trị của thuộc tính đó 

 * search filter: Biểu thức mô tả các loại entry sẽ được giữ lại 
 
 * Danh sách các thuộc tính được giữ lại với mỗi entry 

**Ldapadd**

ldapadd được dùng để add entries vào directory. ldapadd mở một connection đến directory server và xác thực user. Sau đó nó mở file ldif để đọc các giá trị trong đó và thực hiện. Nếu thành công thì các entries được cấu hình trong file ldif sẽ được thêm vào 

Điều kiện: 

 * Entry là parent của entry mới phải tồn tại 
 * DN của entry mới là duy nhất 

Ví dụ: 

    ldapadd -h myhost -p 389 -D "cn=vccloudadmin" -w welcome -f file.ldif 

--> User vccloudadmin xác thực tới directory server myhost, cổng 389. Sau đó lệnh sẽ mở file file.ldif và thêm nội dung của file đó vào trong directory server.  

**Ldapdelete**

ldapdelete xóa một "lá" ra khỏi directory. ldapdelete mở một kết nối tới directory và xác thực user, sau đó nó xóa entries theo yêu cầu. 

Điều kiện: entry phải tồn tại và không có entry con bên trong 

Cú pháp: ldapdelete [options] "entry's DN" 

Ví dụ: 
```
ldapdelete -h myhost -p 389 -D "cn=vccloudadmin" -w welcome "uid=hricard, ou=sales, ou=people, dc=acme, dc=com" 
--> User vccloud admin xác thực tới directory server myhost, cổng 389, mật khẩu welcome, sau đó nó xóa entry có uid=hricard, ou=sales, ou=people, dc=acme, dc=com 
```

**Ldapmodify**

ldapmodify sửa (thay đổi) một hoặc nhiều entries đã tồn tại trong directory. ldapmodify mở một connection tới directory và xác thực user, sau đó nó mở file ldif có chứa sẵn các tham số và các lệnh sửa đổi và thực hiện modify các entries trong directory theo yêu cầu của file ldif đó 

Điều kiện: 

 * entry phải tồn tại 
 * Tất cả các thuộc tính thay đổi đều thành công 
 * Các thao tác cập nhật là được phép đối với user authenticated 

Có 4 kiểu sửa đổi của ldapmodify là: 

 * add: thêm mới một hay nhiều entries 
 * modify: sửa đổi các thuộc tính của các entries đã tồn tại, cũng có thể thêm mới và xóa 
 * delete: xóa một entry đã tồn tại 
 * modrdn: thay đổi RDN của một entry đã tồn tại 

Cú pháp: ldapmodify [options] [-f LDIF-filename] 

Ví dụ: 
```
ldapmodify -h myhost -p 389 -D "cn=vccloudadmin" -w welcome -f hricard.ldif 
```

--> user vccloudadmin được xác thực tới directory server myhost, cổng 389, mật khẩu welcome. Sau đó câu lệnh sẽ mở file hricard.ldif và modify các entries trong directory theo yêu cầu của file ldif đó 

**Ldapmoddn**

ldapmoddn có thể thay đổi RDN của một entry hay di chuyển một entry hoặc thậm chí một subtree tới một vị trí khác trong directory. 

Cú pháp: 
```
ldapmoddn [options] -b "current DN" -R "new RDN" -N "new parents"
```

Ví dụ: 
```
ldapmoddn -h myhost -p 389 -D "cn=vccloudadmin" -w welcome \ 
-b "uid=oball, ou=sales, ou=people, dc=acme, dc=com" \ 
-N "ou=marketing, ou=people, dc=acme, dc=com" 
```

--> Lệnh trên sẽ xác thực user vccloudadmin tới directory server myhost, cổng 389, password welcome. Sau đó nó sẽ tham chiếu tới entry có uid=oball, ou=sales, ou=people, dc=acme, dc=com và sửa entry cha của nó từ ou=sales thành ou=marketing 

Các arguments trong câu lệnh ldap  

|-h   | hostname của directory server   |
|---|---|
|-p |port number của directory server |
| -D |The BIND DN, có nghĩa là user authenticating tới directory server    |
|-w|  Mật khẩu của BIND DN  |
|-W | Phương thức xác thực - 1 hoặc 2 way xác thực SSL   |
|-P |  Mật khẩu cho phương thức xác thực bên trên  |
| -U|  SSL authentication mode: 1 for no authentication 2 for one-way authentication 3 for two-way authentication  |
| -b|  Base DN trong khi tìm kiếm  |
| -s| Search scope của cuộc tìm kiếm base: the entry requested one: the entries just below the requested entry sub: then entire subtree  |
|-f | File ldif chứa các thông tin: thêm, sửa xóa   |
| -R|  New RDN  |
|-N | New parent for an entry or subtree that is moved   |

### 8. Các file cấu hình 

Thư mục cấu hình /etc/ldap/slapd.d/ chứa các file cấu hình cho slapd. Không thực hiện chỉnh sửa các file này trực tiếp mà phải sử dụng các ldap-utilities 
```
/etc/ldap/slapd.d/ 
    /etc/ldap/slapd.d/cn=config.ldif 
    /etc/ldap/slapd.d/cn=config 
    /etc/ldap/slapd.d/cn=config/cn=schema 
    /etc/ldap/slapd.d/cn=config/cn=schema/cn={1}cosine.ldif 
    /etc/ldap/slapd.d/cn=config/cn=schema/cn={0}core.ldif 
    /etc/ldap/slapd.d/cn=config/cn=schema/cn={2}nis.ldif 
    /etc/ldap/slapd.d/cn=config/cn=schema/cn={3}inetorgperson.ldif 
    /etc/ldap/slapd.d/cn=config/cn=module{0}.ldif 
    /etc/ldap/slapd.d/cn=config/olcDatabase={0}config.ldif 
    /etc/ldap/slapd.d/cn=config/olcDatabase={-1}frontend.ldif 
    /etc/ldap/slapd.d/cn=config/olcDatabase={1}mdb.ldif 
    /etc/ldap/slapd.d/cn=config/olcBackend={0}mdb.ldif 
    /etc/ldap/slapd.d/cn=config/cn=schema.ldif 
``` 

Khi sử dụng ldapsearch để tìm kiếm các file config này: 

```
sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config dn 
dn: cn=config 
dn: cn=module{0},cn=config 
dn: cn=schema,cn=config 
dn: cn={0}core,cn=schema,cn=config 
dn: cn={1}cosine,cn=schema,cn=config 
dn: cn={2}nis,cn=schema,cn=config 
dn: cn={3}inetorgperson,cn=schema,cn=config 
dn: olcBackend={0}mdb,cn=configdn: olcDatabase={-1}frontend,cn=configdn: olcDatabase={0}config,cn=config 
dn: olcDatabase={1}mdb,cn=config 
``` 

### 9. REPLICATION 
#### a. LDAP sysnc replication - Cơ chế replication 

LDAP sync replication engine (syncrepl) là một công cụ replicate phía consumer cho phép LDAP consumer server duy trì một shadow copy của DIT (cây thư mục). Nó tạo ra và duy trì bản sao (replica) bằng cách kết nối với provider để thực hiện việc tải nội dung DIT cập nhật dựa vào việc polling nội dung định kỳ hoặc cập nhật ngay khi có sự thay đổi nội dung 

Syncrepl sử dụng giao thức đồng bộ hóa nội dung LDAP (LDAP content synchronization protocol - LDAP sync) làm giao thức đồng bộ hóa bản sao (replica synchronization protocol). LDAP sync cung cấp một bản sao trạng thái hỗ trợ đồng bộ hóa cả 2 cách là pull-based và push-base. Trong pull-based replication, consumer kiểm tra định kỳ provider để cập nhật, còn trong push-based replication, consumer lắng nghe các bản cập nhật được provider gửi đến một cách real-time. Giao thức LDAP sysc không yêu cầu lưu trữ lại lịch sử nên việc cập nhật này không cần được log lại 

Với syncrepl, một consumer server có thể tạo ra một bản sao mà không cần thay đổi cấu hình của provider, cũng không cần khởi động lại dịch vụ.  

### b. Deployment Alternatives - Giải pháp triển khai 

**Delta-syncrepl replication**

Nhược điểm của giao thức đồng bộ hóa nội dung LDAP là cơ chế nhân bản dựa trên đối tượng, tức là khi có bất kỳ giá trị thuộc tính nào trong một đối tượng được sao chép đến thay đổi trên provider thì trên consumer server sẽ tìm nạp và xử lý TOÀN BỘ đối tượng, cả những thuộc tính thay đổi và giữ nguyên. Điều này sẽ gây lãng phí băng thông truyền tải vì vẫn phải sync cả những thứ không thay đổi chẳng để làm gì, có chăng chỉ là duy trì thứ tự chính xác của các thuộc tính trên đối tượng mà thôi. 

--> giải pháp là delta-syncrepl 

Delta-syncrepl duy trì một changelog của toàn bộ hoặc một phần (tùy chọn) của cơ sở dữ liệu, tách biệt với provider. Consumer sẽ dựa vào changelog này để áp dụng chỉ những thay đổi cần thiết. 

**N-way multi-master Replication**

Multi-master replication là một kỹ thuật tạo bản sao bằng cách sử dụng syncrepl để sao chép dữ liệu tới nhiều master - directory server 

Nếu một provider nào bị fails, các providers khác sẽ tiếp tục chấp nhận các updates 

Ngăn ngừa single point failure, tốt cho HA và Failover 

Provider có thể ở bất cứ đâu - toàn cầu --> dữ liệu phân tán 

**MirrorMode replication**

MirrorMode là một cấu hình lai cung cấp cả 2 ưu điểm là vừa có tính nhất quán của single-master replication trong khi vẫn mang lại tính HA của multi-master replication. Trong mirrormode, mỗi cái trong 2 providers được thiết lập để nó sao chép từ cái còn lại, trong đó có 1 providers là chính, toàn bộ request sẽ được gửi đến provider này, cái còn lại chỉ được sử dụng khi cái đầu tiên fail. khi đó toàn bộ request lại gửi đến provider thứ 2. Khi provider đầu tiên được sửa và khởi động lại, nó sẽ tự động cập nhật và resync những thay đổi từ provider thứ 2 

**Proxy Mode Syncrepl - Replacing Slurpd**

Từ phiên bản slapd 2.4, phần này đã bị loại bỏ 

## II. CÀI ĐẶT VÀ CẤU HÌNH 
### 1. Cài đặt và cấu hình LDAP server với openLDAP 
#### a. Cài đặt slapd và ldap-utils trên openldap server 

**Cài đặt**

slapd (openldap standalone server): Tạo ra một standalone directory service và bao gồm cả slurpd replication server 

    root@oldap-master:~# apt-get install -y slapd 

ldap-utils: Chứa các tiện ích dùng để truy cập ldap server local hoặc remote. Utilities này cũng chứa toàn bộ các chương trình cần thiết (client required programs) để truy cập các ldap server 

    root@oldap-master:~# apt-get install -y ldap-utils 

Mặc định khi cài đặt slapd, hệ thống sẽ yêu cầu người dùng đặt một mật khẩu và tự tạo ra user mặc định là admin sử dụng mật khẩu này, có quyền truy cập cao nhất. Chúng ta có thể cài đặt lại các thuộc tính này bằng cách sử dụng lệnh dpkg-reconfigure slapd 

**Thử thêm các entries vào database**

Thử thêm các entries sau: 

 * 1 node People để quản lý các users 
 * 1 node Groups để quản lý các team nhân sự 
 * 1 Group là teamSVR 
 * 1 user là vantuanpham 

Tạo một file add_content.ldif có nội dung sau 

```
dn: ou=People,dc=vccloud,dc=vn 
objectClass: organizationalUnit 
ou: People 
  
dn: ou=Groups,dc=vccloud,dc=vn 
objectClass: organizationalUnit 
ou: Groups 
  
dn: cn=teamSVR,ou=Groups,dc=vccloud,dc=vn 
objectClass: posixGroup 
cn: teamSVR 
gidNumber: 5000 
  
dn: uid=vantuanpham,ou=People,dc=vccloud,dc=vn 
objectClass: inetOrgPerson 
objectClass: posixAccount 
objectClass: shadowAccount 
uid: vantuanpham 
sn: Pham 
givenName: Van Tuan 
cn: vantuanpham 
displayName: Van Tuan Pham 
uidNumber: 10000 
gidNumber: 5000 
userPassword: 123456 
gecos: Van Tuan Pham 
loginShell: /bin/bash 
homeDirectory: /home/vantuanpham 
``` 

Sử dụng lệnh sau để add nội dung vào database 

```
root@oldapsvr:~# ldapadd -x -D cn=admin,dc=vccloud,dc=vn -W -f add_content.ldif  
Enter LDAP Password:  
adding new entry "ou=People,dc=vccloud,dc=vn" 
  
adding new entry "ou=Groups,dc=vccloud,dc=vn" 
  
adding new entry "cn=teamSVR,ou=Groups,dc=vccloud,dc=vn" 
  
adding new entry "uid=vantuanpham,ou=People,dc=vccloud,dc=vn" 
``` 

Check lại bằng cách sử dụng ldapsearch 

```
root@oldapsvr:~# ldapsearch -x -LLL -b dc=vccloud,dc=vn dn 
dn: dc=vccloud,dc=vn 
  
dn: cn=admin,dc=vccloud,dc=vn 
  
dn: ou=People,dc=vccloud,dc=vn 
  
dn: ou=Groups,dc=vccloud,dc=vn 
  
dn: cn=teamSVR,ou=Groups,dc=vccloud,dc=vn 
  
dn: uid=vantuanpham,ou=People,dc=vccloud,dc=vn 
```
### 2. REPLICATION 
#### 1. Syncrepl 
Trên provider: 

Thiết lập file slapd.conf 

Có 2 tùy chọn chính khi cấu hình syncrepl là checkpoint và sessionlog. 

```
syncprov-checkpoint <ops> <minutes> 
syncprov-sessionlog <ops> 
```

Trong đó ops là số hoạt động có thể cập nhật tối đa mà sessionlog sẽ ghi lại trong 1 phiên check. Còn minutes là số phút kể từ lần check cuối cùng. Khi sau số ops hành vi hoặc sau số phút đó thì checkpoint mới được xác lập. 
 
Mặc định, tùy chọn reloadhint bị để cờ fail, khi config syncrepl thì ta cần bật nó lên. 

    syncprov-reloadhint true 

Ví dụ config slapd.conf 

```
database mdb 
        maxsize 1073741824 
        suffix dc=vccloud,dc=vn 
        rootdn dc=vccloud,dc=vn 
        directory /var/ldap/db 
        index objectclass,entryCSN,entryUUID eq 
  
        overlay syncprov 
        syncprov-checkpoint 100 10 
        syncprov-sessionlog 100 
```

Trên consumer 

Ví dụ: 
```
database mdb 
        maxsize 1073741824 
        suffix dc=Vccloud,dc=vn 
        rootdn dc=Vccloud,dc=vn 
        directory /var/ldap/db 
        index objectclass,entryCSN,entryUUID eq 
  
        syncrepl rid=123 
                provider=ldap://myhost:389 
                type=refreshOnly 
                interval=01:00:00:00 
                searchbase="dc=vccloud,dc=vn" 
                filter="(objectClass=organizationalPerson)" 
                scope=sub 
                attrs="cn,sn,ou,telephoneNumber,title,l" 
                schemachecking=off 
                bindmethod=simple 
                binddn="cn=syncuser,dc=vccloud,dc=vn" 
                credentials=secret 
```
Theo config bên trên, consumer sẽ connect tới provider myhost, cổng 389 để thực hiện sync mỗi lần một ngày. Sync user được sử dụng là cn=syncuser,dc=vccloud,dc=vn. Ở đây nên sử dung rootdn để có quyền đọc ghi và thực hiện các ldaputils cao nhất. 
Ngoài ra có thể sử dụng filter để cấu hình chỉ đồng bộ một phần của cây thư mục, theo thuộc tính filter bên trên thì consumer sẽ sync toàn bộ phần có objectClass là organizationalPerson bắt đầu từ rootdn, có các thuộc tính (attributes là là cn,sn,ou,telephonenumber,title và l) 

--> không yêu cầu khởi động lại dịch vụ 
 
#### b. Delta-syncrepl 

Delta-syncrepl yêu cầu config trên cả provider và consumer 

Trên provider 

```
# Give the replica DN unlimited read access.  This ACL needs to be 
     # merged with other ACL statements, and/or moved within the scope 
     # of a database.  The "by * break" portion causes evaluation of 
     # subsequent rules.  See slapd.access(5) for details. 
     access to * 
        by dn.base="cn=replicator,dc=symas,dc=com" read 
        by * break 
  
     # Set the module path location 
     modulepath /opt/symas/lib/openldap 
  
     # Load the hdb backend 
     moduleload back_hdb.la 
  
     # Load the accesslog overlay 
     moduleload accesslog.la 
  
     #Load the syncprov overlay 
     moduleload syncprov.la 
  
     # Accesslog database definitions 
     database hdb 
     suffix cn=accesslog 
     directory /db/accesslog 
     rootdn cn=accesslog 
     index default eq 
     index entryCSN,objectClass,reqEnd,reqResult,reqStart 
  
     overlay syncprov 
     syncprov-nopresent TRUE 
     syncprov-reloadhint TRUE 
  
     # Let the replica DN have limitless searches 
     limits dn.exact="cn=replicator,dc=symas,dc=com" time.soft=unlimited time.hard=unlimited size.soft=unlimited size.hard=unlimited 
  
     # Primary database definitions 
     database hdb 
     suffix "dc=symas,dc=com" 
     rootdn "cn=manager,dc=symas,dc=com" 
  
     ## Whatever other configuration options are desired 
  
     # syncprov specific indexing 
     index entryCSN eq 
     index entryUUID eq 
  
     # syncrepl Provider for primary db 
     overlay syncprov 
     syncprov-checkpoint 1000 60 
  
     # accesslog overlay definitions for primary db 
     overlay accesslog 
     logdb cn=accesslog 
     logops writes 
     logsuccess TRUE 
     # scan the accesslog DB every day, and purge entries older than 7 days 
     logpurge 07+00:00 01+00:00 
  
     # Let the replica DN have limitless searches 
     limits dn.exact="cn=replicator,dc=symas,dc=com" time.soft=unlimited time.hard=unlimited size.soft=unlimited size.hard=unlimited 
``` 

Trên consumer 

```
# Replica database configuration 
     database hdb 
     suffix "dc=symas,dc=com" 
     rootdn "cn=manager,dc=symas,dc=com" 
  
     ## Whatever other configuration bits for the replica, like indexing 
     ## that you want 
  
     # syncrepl specific indices 
     index entryUUID eq 
  
     # syncrepl directives 
     syncrepl  rid=0 
               provider=ldap://ldapmaster.symas.com:389 
               bindmethod=simple 
               binddn="cn=replicator,dc=symas,dc=com" 
               credentials=secret 
               searchbase="dc=symas,dc=com" 
               logbase="cn=accesslog" 
               logfilter="(&(objectClass=auditWriteObject)(reqResult=0))" 
               schemachecking=on 
               type=refreshAndPersist 
               retry="60 +" 
               syncdata=accesslog 
  
     # Refer updates to the master 
     updateref               ldap://ldapmaster.symas.com 
 
``` 
#### c. N-Way multi-master 

Cấu hình thông qua cn=config, ví dụ cấu hình 3 node multi-master 

Trên node1 cấu hình: 

```
dn: cn=config 
     objectClass: olcGlobal 
     cn: config 
     olcServerID: 1 
  
     dn: olcDatabase={0}config,cn=config 
     objectClass: olcDatabaseConfig 
     olcDatabase: {0}config 
     olcRootPW: secret 
```

Trên node2 và node3 cấu hình tương tự nhưng bắt buộc phải khác olcServerID 

```
dn: cn=config 
     objectClass: olcGlobal 
     cn: config 
     olcServerID: 2 
  
     dn: olcDatabase={0}config,cn=config 
     objectClass: olcDatabaseConfig 
     olcDatabase: {0}config 
     olcRootPW: secret 
```

Cấu hình syncrepl là provider (vì tất cả đều là master) 

```
dn: cn=module,cn=config 
     objectClass: olcModuleList 
     cn: module 
     olcModulePath: /usr/local/libexec/openldap 
     olcModuleLoad: syncprov.la 
```

Trên cả 3 node, sửa file hosts, thêm vào IP của các node bằng các dòng 

```
45.124.94.125 OLDAP1 
45.124.94.98 OLDAP2 
45.124.95.42 OLDAP3 
``` 

Trên cả 3 node,và tạo file ldif như sau và sử dụng ldapadd dưới quyền admin 

```
dn: cn=config 
     changetype: modify 
     replace: olcServerID 
     olcServerID: 1 OLDAP1 
     olcServerID: 2 OLDAP2 
     olcServerID: 3 OLDAP3 
  
     dn: olcOverlay=syncprov,olcDatabase={0}config,cn=config 
     changetype: add 
     objectClass: olcOverlayConfig 
     objectClass: olcSyncProvConfig 
     olcOverlay: syncprov 
  
     dn: olcDatabase={0}config,cn=config 
     changetype: modify 
     add: olcSyncRepl 
     olcSyncRepl: rid=001 provider=OLDAP1 binddn="cn=config" bindmethod=simple 
       credentials=secret searchbase="cn=config" type=refreshAndPersist 
       retry="5 5 300 5" timeout=1 
     olcSyncRepl: rid=002 provider=OLDAP2 binddn="cn=config" bindmethod=simple 
       credentials=secret searchbase="cn=config" type=refreshAndPersist 
       retry="5 5 300 5" timeout=1 
     olcSyncRepl: rid=003 provider=OLDAP3 binddn="cn=config" bindmethod=simple 
       credentials=secret searchbase="cn=config" type=refreshAndPersist 
       retry="5 5 300 5" timeout=1 
     - 
     add: olcMirrorMode 
     olcMirrorMode: TRUE 
``` 
#### d. MIRROR MODE 

Cấu hình giống với phần syncrepl. Cấu hình mirror mode rất đơn giản, các node giống y như nhau chỉ khác mỗi serverID 

Thử cấu hình 2 node - mirror mode 

Trên node1 

```
# Global section 
       serverID    1 
       # database section 
  
       # syncrepl directive 
       syncrepl      rid=001 
                     provider=ldap://OLDAP1 
                     bindmethod=simple 
                     binddn="cn=mirrormode,dc=vccloud,dc=vn" 
                     credentials=mirrormode 
                     searchbase="dc=vccloud,dc=vn" 
                     schemachecking=on 
                     type=refreshAndPersist 
                     retry="60 +" 
  
       mirrormode on 
``` 

Trên node2 

```
# Global section 
       serverID    2 
       # database section 
  
       # syncrepl directive 
       syncrepl      rid=001 
                     provider=ldap://OLDAP2 
                     bindmethod=simple 
                     binddn="cn=mirrormode,dc=vccloud,dc=vn" 
                     credentials=mirrormode 
                     searchbase="dc=vccloud,dc=vn" 
                     schemachecking=on 
                     type=refreshAndPersist 
                     retry="60 +" 
  
       mirrormode on 
```
