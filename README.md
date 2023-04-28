# SMTP MTA Strict Transport Security (MTA-STS) 
An MTA-STS implementation based on [rfc8461](https://tools.ietf.org/html/rfc8461).

## Installation

Follow the [instructions](https://docs.halon.io/manual/comp_install.html#installation) in our manual to add our package repository and then run the below command.

### Ubuntu

```
apt-get install halon-extras-mta-sts
```

### RHEL

```
yum install halon-extras-mta-sts
```

## Usage

You can test it by running:

```
import { mta_sts } from "extras://mta-sts";

$domain = "gmail.com";
$mtasts = mta_sts($domain);
if (is_array($mtasts))
{
	if ($mtasts["error"])
		echo "Bad MTA-STS for $domain";
	if ($mtasts["policy"]["mode"] == "enforce")
		smtp_lookup_rcpt([
			"mx_include" => $mtasts["policy"]["mx"],
			"tls" => "dane_fallback_require_verify",
			"tls_sni" => true,
			"tls_verify_host" => true,
			"tls_default_ca" => true,
			"tls_protocols" => "!SSLv2,!SSLv3,!TLSv1,!TLSv1.1"
		], "", "test@$domain");
}
else echo "No MTA-STS for $domain";
```

and it should normally be used in the [pre-delivery script](https://docs.halon.io/hsl/archive/master/predelivery.html) like

```
import { mta_sts } from "extras://mta-sts";
//import { tls_rpt } from "extras://tls-rpt"; // not available yet

$mtasts = mta_sts($message["recipientaddress"]["domain"]);
if (is_array($mtasts))
{
	$context["sts"] = $mtasts;
	if ($mtasts["error"])
	{
//		if (tls_rpt_fetch_dnstxt($message["recipientaddress"]["domain"]))
//		{
//			tls_rpt([], $message, $mtasts);
//		}
		Queue(["reason" => "Bad MTA-STS:" . $mtasts["error"]]);
	}
	if ($mtasts["policy"]["mode"] == "enforce")
		Try([
			"mx_include" => $mtasts["policy"]["mx"],
			"tls" => "dane_fallback_require_verify",
			"tls_sni" => true,
			"tls_verify_host" => true,
			"tls_default_ca" => true,
			"tls_protocols" => "!SSLv2,!SSLv3,!TLSv1,!TLSv1.1"
		]);
  ...
```
