### Biên dịch trong môi trường Visual Studio

Chạy các lệnh sau trong Terminal của Visual Studio:

```bash
mkdir build
cd build
cmake .. -G "NMake Makefiles" -DWIN32=ON
nmake
```

### Tạo Cert và Key

```bash
gmssl sm2keygen -pass P@ssw0rd -out sm2_root_ca_key.pem
gmssl certgen -C VN -ST HaNoi -L HaDong -O GmSSL -OU Test -CN "GmSSL SM2 Test Root CA" -days 3650 -key sm2_root_ca_key.pem -pass P@ssw0rd -out sm2_root_ca_cert.pem -key_usage keyCertSign -key_usage cRLSign -ca

gmssl sm2keygen -pass P@ssw0rd -out sm2_tlcp_ca_key.pem
gmssl reqgen -C VN -ST HaNoi -L HaDong -O GmSSL -OU Test -CN "GmSSL SM2 TLCP CA" -key sm2_tlcp_ca_key.pem -pass P@ssw0rd -out sm2_tlcp_ca_req.pem
gmssl reqsign -in sm2_tlcp_ca_req.pem -days 1825 -key_usage keyCertSign -key_usage cRLSign -path_len_constraint 0 -cacert sm2_root_ca_cert.pem -key sm2_root_ca_key.pem -pass P@ssw0rd -out sm2_tlcp_ca_cert.pem -ca

gmssl sm2keygen -pass P@ssw0rd -out sm2_tlcp_server_sign_key.pem
gmssl reqgen -C VN -ST HaNoi -L HaDong -O GmSSL -OU Test -CN "GmSSL SM2 TLCP Server" -key sm2_tlcp_server_sign_key.pem -pass P@ssw0rd -out sm2_tlcp_server_sign_req.pem
gmssl reqsign -in sm2_tlcp_server_sign_req.pem -days 365 -key_usage digitalSignature -ext_key_usage serverAuth -subject_dns_name localhost -cacert sm2_tlcp_ca_cert.pem -key sm2_tlcp_ca_key.pem -pass P@ssw0rd -out sm2_tlcp_server_sign_cert.pem
gmssl sm2keygen -pass P@ssw0rd -out sm2_tlcp_server_enc_key.pem
gmssl reqgen -C VN -ST HaNoi -L HaDong -O GmSSL -OU Test -CN "GmSSL SM2 TLCP Server" -key sm2_tlcp_server_enc_key.pem -pass P@ssw0rd -out sm2_tlcp_server_enc_req.pem
gmssl reqsign -in sm2_tlcp_server_enc_req.pem -days 365 -key_usage keyEncipherment -ext_key_usage serverAuth -subject_dns_name localhost -cacert sm2_tlcp_ca_cert.pem -key sm2_tlcp_ca_key.pem -pass P@ssw0rd -out sm2_tlcp_server_enc_cert.pem

copy /b sm2_tlcp_server_sign_key.pem + sm2_tlcp_server_enc_key.pem sm2_tlcp_server_keys.pem
copy /b sm2_tlcp_server_sign_cert.pem + sm2_tlcp_server_enc_cert.pem + sm2_tlcp_ca_cert.pem sm2_tlcp_server_certs.pem
```

### Chạy demo

Server:

```bash
gmssl tlcp_server -port 4431 -cert sm2_tlcp_server_certs.pem -key sm2_tlcp_server_keys.pem -pass P@ssw0rd -cipher_suite TLS_ECC_SM4_CBC_SM3 -www -verbose
```

Client:

```bash
gmssl tlcp_client -host 127.0.0.1 -port 4431 -server_name localhost -cacert sm2_root_ca_cert.pem -cipher_suite TLS_ECC_SM4_CBC_SM3 -get /index.html
```