# SRI - sistema.sol 

## DESCRIPTION

1. Set the `dnssec-validation` option to `yes`
2. Configure `tierra` and `venus` to be the default name servers (file /etc/resolv.conf).
3. The name server will have authority over the `sistema.sol` domain.
4. Allow recursive queries only from computers in the `127.0.0.0/8` and `192.168.57.0/24` networks.
5. The master server will be `tierra.sistema.sol` and will have authority over the forward and reverse zones.
6. The slave server will be `venus.sistema.sol` and will have `tierra.sistema.sol` as its master.
7. The time in cache for negative responses from the zones (forward and reverse) will be two hours (specified in seconds).
8. Forward queries that the server is not authorized for to the DNS server `208.67.222.222`.
9. Configure the following aliases:
   - `ns1.sistema.sol.` will be an alias for `tierra.sistema.sol`.
   - `ns2.sistema.sol.` will be an alias for `venus.sistema.sol`.
10. `mail.sistema.sol.` will be an alias for `marte.sistema.sol`.
11. The `marte.sistema.sol.` system will act as the mail server for the `sistema.sol` mail domain.

## BIND CONFIGURATION

### named.conf.options

```conf
acl SistemaSol {
	127.0.0.0/8;
	192.168.57.0/24;
};

options {
	directory "/var/cache/bind";

	forwarders {
	 	208.67.222.222;

	};
	
	allow-transfer { 192.168.57.102; };
	
	listen-on port 53 { 192.168.57.103; };
	
	recursion yes;
	allow-recursion { SistemaSol; }; 

	dnssec-validation yes;
};

```

- `directory`: here we insert the path of the cache
- `acl`: here we define the network our DNS server will trust
- `forwarders`: we will sent this server to the request if we don't have the IP in our domain
- `allow-transfer { 192.168.57.102; }`: *this option will be only on tierra*  here we specified the IP of venus, which our slave server.
- `recursion`: allow recursive requests. If we don't have the IP we are being asked, we will ask the client to look for it in other DNS servers
- `listen-on`: here we define where will be our server listening for requests. *the IP will change depending on the server we are (tierra or venus)*

### named.conf.local - Tierra

```conf
zone "sistema.sol" {
	type master;
	file "/var/lib/bind/sistema.sol.dns";
};

zone "57.168.192.in-addr.arpa" {
	type master;
	file "/var/lib/bind/sistema.sol.rev";
};
```

- In this file we will specify the role of the server (in the given zone), and where are the zone files located.


### named.conf.local - Venus

```conf
zone "sistema.sol" {
	type slave;
	masters { 192.168.57.103; };
	file "/var/lib/bind/sistema.sol.dns";
};

zone "57.168.192.in-addr.arpa" {
	type slave;
	masters { 192.168.57.103; };
	file "/var/lib/bind/sistema.sol.rev";
};
```

- Same here. But we can see how the role of the server has changed. In the case of the slave, we must specify the IP of the master.

## ZONES CONFIGURATION

### sistema.sol.dns

```conf
$TTL	86400
@	IN	SOA	tierra.sistema.sol. elena.sistema.sol. (
			      1				; Serial
			 604800				; Refresh
			  86400				; Retry
			2419200				; Expire
			  7200 )			; Negative Cache TTL
;

; NAMESERVERS
@						IN	NS			venus.sistema.sol.
@						IN	NS			tierra.sistema.sol.

tierra.sistema.sol.		IN	A			192.168.57.103
venus.sistema.sol.		IN	A			192.168.57.102
mercurio.sistema.sol.	IN	A			192.168.57.101
marte.sistema.sol.		IN	A			192.168.57.104


; MAILSERVERS
sistema.sol.			IN	MX 10		marte.sistema.sol.

; ALIASES
ns1.sistema.sol.		IN	CNAME		tierra.sistema.sol.
ns2.sistema.sol.		IN	CNAME		venus.sistema.sol.
mail.sistema.sol.		IN	CNAME		marte.sistema.sol.

```

- This will be the file where we will store the info related to the zones. 

### sistema.sol.rev

```conf
$TTL	86400
@	IN	SOA	tierra.sistema.sol. 	elena.sistema.sol. (
			      1					; Serial
			 604800					; Refresh
			  86400					; Retry
			2419200					; Expire
			  7200 )				; Negative Cache TTL
;


; NAMESERVERS
@			IN	NS	tierra.sistema.sol.
@			IN	NS	venus.sistema.sol.

; IP TO NAMESERVERS 
103			IN	PTR		tierra.sistema.sol.
101			IN	PTR		mercurio.sistema.sol.
102			IN	PTR		venus.sistema.sol.
104			IN	PTR		marte.sistema.sol.
```
- This is the reverse zone. It will store IP to name translations 
