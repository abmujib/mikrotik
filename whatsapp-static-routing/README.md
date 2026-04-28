# WHATSAPP STATIC ROUTING MIKROTIK RoS7

```shell
/ip firewall address-list add address=192.168.0.0/16 list=LOCAL comment="LOCAL IP"
/ip firewall address-list add address=172.16.0.0/12 list=LOCAL comment="LOCAL IP"
/ip firewall address-list add address=10.0.0.0/8 list=LOCAL comment="LOCAL IP"
```

```shell
/ip firewall raw
add action=add-dst-to-address-list address-list=whatsapp_list address-list-timeout=1d chain=prerouting comment="WhatsApp Routing" content=wa.me dst-address-list=!LOCAL src-address-list=LOCAL
add action=add-dst-to-address-list address-list=whatsapp_list address-list-timeout=1d chain=prerouting comment="WhatsApp Routing" content=.whatsapp. dst-address-list=!LOCAL src-address-list=LOCAL
```

```shell
/ip firewall address-list
add address=whatsapp.com list=whatsapp_list
add address=v.whatsapp.com list=whatsapp_list
add address=account.whatsapp.com list=whatsapp_list
add address=chat.whatsapp.com list=whatsapp_list
add address=faq.whatsapp.com list=whatsapp_list
add address=web.whatsapp.com list=whatsapp_list
add address=www.whatsapp.com list=whatsapp_list
add address=api.whatsapp.com list=whatsapp_list
add address=autodiscover.whatsapp.com list=whatsapp_list
add address=b.whatsapp.com list=whatsapp_list
add address=blog.whatsapp.com list=whatsapp_list
add address=business.whatsapp.com list=whatsapp_list
add address=wa.me list=whatsapp_list
```

```shell
/routing table add name="to_ISP1" fib comment="r_whatsapp"
```

```shell
/ip route add check-gateway=ping distance=1 gateway="0.0.0.0" routing-table="to_ISP1" comment="r_whatsapp"
```

```shell
/ip firewall mangle add action=mark-routing chain=prerouting src-address-list=LOCAL dst-address-list="whatsapp_list" new-routing-mark="to_ISP1" comment="WhatsApp Routing" passthrough=no place-before=*0
```