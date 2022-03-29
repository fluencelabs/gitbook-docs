# cUrl As A Service

### Overview

[Curl](https://curl.se) is a widely available and used command-line tool to receive or send data using URL syntax. Chances are, you probably just used it when you set up your Fluence development environment. For Fluence services to be able to interact with the world, cUrl is one option to facilitate https calls. Since Fluence modules are Wasm IT modules, cUrl cannot not be a service intrinsic. Instead, the curl command-line tool needs to be made available and accessible at the node level. And for Fluence services to be able to interact with Curl, we need to code a cUrl adapter taking care of the mounted (cUrl) binary.

### Adapter Construction

The work for the cUrl adapter has been fundamentally done and is exposed by the Fluence Rust SDK. As a developer, the task remaining is to instantiate the adapter in the context of the module and services scope. The following code [snippet](https://github.com/fluencelabs/marine/tree/master/examples/url-downloader/curl\_adapter) illustrates the implementation requirement. It is a part of a larger service, [url-downloader](https://github.com/fluencelabs/marine/tree/master/examples/url-downloader).

```rust
use marine_rs_sdk::marine;

use marine_rs_sdk::WasmLoggerBuilder;
use marine_rs_sdk::MountedBinaryResult;

pub fn main() {
    WasmLoggerBuilder::new().build().unwrap();
}

#[marine]
pub fn download(url: String) -> String {
    log::info!("download called with url {}", url);

    let result = unsafe { curl(vec![url]) };
    String::from_utf8(result.stdout).unwrap()
}

#[marine]
#[link(wasm_import_module = "host")]
extern "C" {
    fn curl(cmd: Vec<String>) -> MountedBinaryResult;
}
```

with the following dependencies necessary in the Cargo.toml:

```rust
marine-rs-sdk = { version = "=0.6.11", features = ["logger"] }
log = "0.4.8"
```

We are basically linking the [external](https://doc.rust-lang.org/std/keyword.extern.html) cUrl binary and are exposing access to it as a `marine` interface called download.

### Code References

* [Mounted binaries](https://github.com/fluencelabs/fce/blob/c559f3f2266b924398c203a45863ebf2fb9252ec/fluence-faas/src/host\_imports/mounted\_binaries.rs)
* [cUrl](https://github.com/curl/curl)

### Service Construction

In order to create a valid Fluence service, a service configuration is required.

```
modules_dir = "target/wasm32-wasi/release"

[[module]]
    name = "curl_adapter"
    logger_enabled = true

    [module.mounted_binaries]
    curl = "/usr/bin/curl"
```

We are specifying the location of the Wasm file, the import name of the Wasm file, some logging housekeeping, and the mounted binary reference with the command-line call information.

### Remote Service Creation

```bash
cargo new curl_adapter
cd curl_adapter
# copy the above rust code into src/main
# copy the specified dependencies into Cargo.toml
# copy the above service configuration into Config.toml

marine build --release
```

You should have the Fluence module cur-service.wasm in `target/wasm32-wasi/release` . We can test our service with `mrepl`.

### Service Testing

Running the REPL, we use the `interface` command to list all available interfaces and the `call` command to run a method. For our purposes, we furnish the [https://duckduckgo.com/?q=Fluence+Labs](https://duckduckgo.com/?q=Fluence+Labs) url to give the the curl adapter a workout.

```bash
mrepl Config.toml
Welcome to the Marine REPL (version 0.9.1)
Minimal supported versions
  sdk: 0.6.0
  interface-types: 0.20.0

app service was created with service id = ee63d3ae-304a-42bb-9a2d-4a4dc056a68b
elapsed time 76.5175ms

1> interface
Loaded modules interface:

curl_adapter:
  fn download(url: string) -> string

2> call curl_adapter download ["https://duckduckgo.com/?q=Fluence+Labs"]
result: String("<!DOCTYPE html><html lang=\"en-US\" class=\"no-js has-zcm  no-theme \"><head><meta name=\"description\" content=\"DuckDuckGo. Privacy, Simplified.\"><meta http-equiv=\"content-type\" content=\"text/html; charset=utf-8\"><title>Fluence Labs at DuckDuckGo</title><link rel=\"stylesheet\" href=\"/s1997.css\" type=\"text/css\"><link rel=\"stylesheet\" href=\"/r1997.css\" type=\"text/css\"><meta name=\"robots\" content=\"noindex,nofollow\"><meta name=\"referrer\" content=\"origin\"><meta name=\"apple-mobile-web-app-title\" content=\"Fluence Labs\"><link rel=\"preconnect\" href=\"https://links.duckduckgo.com\"><link rel=\"preload\" href=\"/font/ProximaNova-Reg-webfont.woff2\" as=\"font\" type=\"font/woff2\" crossorigin=\"anonymous\" /><link rel=\"preload\" href=\"/font/ProximaNova-Sbold-webfont.woff2\" as=\"font\" type=\"font/woff2\" crossorigin=\"anonymous\" /><link rel=\"shortcut icon\" href=\"/favicon.ico\" type=\"image/x-icon\" /><link id=\"icon60\" rel=\"apple-touch-icon\" href=\"/assets/icons/meta/DDG-iOS-icon_60x60.png?v=2\"/><link id=\"icon76\" rel=\"apple-touch-icon\" sizes=\"76x76\" href=\"/assets/icons/meta/DDG-iOS-icon_76x76.png?v=2\"/><link id=\"icon120\" rel=\"apple-touch-icon\" sizes=\"120x120\" href=\"/assets/icons/meta/DDG-iOS-icon_120x120.png?v=2\"/><link id=\"icon152\" rel=\"apple-touch-icon\" sizes=\"152x152\" href=\"/assets/icons/meta/DDG-iOS-icon_152x152.png?v=2\"/><link rel=\"image_src\" href=\"/assets/icons/meta/DDG-icon_256x256.png\"/><script type=\"text/javascript\">var ct,fd,fq,it,iqa,iqm,iqs,iqp,iqq,qw,dl,ra,rv,rad,r1hc,r1c,r2c,r3c,rfq,rq,rds,rs,rt,rl,y,y1,ti,tig,iqd,shfl,shrl,locale,settings_js_version='s2481.js',is_twitter='',rpl=1;fq=0;fd=1;it=0;iqa=0;iqbi=0;iqm=0;iqs=0;iqp=0;iqq=0;qw=2;dl='en';ct='RU';iqd=0;r1hc=0;r1c=0;r3c=0;rq='Fluence%20Labs';rqd=\"Fluence Labs\";rfq=0;rt='';ra='';rv='';rad='';rds=30;rs=0;spice_version='2000';spice_paths='{}';locale='en_US';settings_url_params={};rl='us-en';shfl=0;shrl='us-en';rlo=0;df='';ds='';sfq='';iar='';vqd='3-167080984343175066141708440165596583146-24182048638364464238089820915205130627';safe_ddg=0;show_covid=0;</script><meta name=\"viewport\" content=\"width=device-width, initial-scale=1\" /><meta name=\"HandheldFriendly\" content=\"true\" /><meta name=\"apple-mobile-web-app-capable\" content=\"no\" /></head><body class=\"body--serp\"><input id=\"state_hidden\" name=\"state_hidden\" type=\"text\" size=\"1\"><!-- Ignore this input please --><div id=\"spacing_hidden_wrapper\"><div id=\"spacing_hidden\"></div></div><script type=\"text/javascript\" src=\"/lib/l123.js\"></script><script type=\"text/javascript\" src=\"/locale/en_US/duckduckgo14.js\"></script><script type=\"text/javascript\" src=\"/util/u572.js\"></script><script type=\"text/javascript\" src=\"/d2984.js\"></script><div class=\"site-wrapper  js-site-wrapper\"><div class=\"welcome-wrap js-welcome-wrap\"></div><div id=\"header_wrapper\" class=\"header-wrap js-header-wrap\"><div id=\"header\" class=\"header  cw\"><div class=\"header__search-wrap\"><a tabindex=\"-1\" href=\"/\" class=\"header__logo-wrap js-header-logo\"><span class=\"header__logo js-logo-ddg\">DuckDuckGo</span></a><div class=\"header__content  header__search\"><form id=\"search_form\" class=\"search--adv  search--header js-search-form\" name=\"x\" action=\"/\"><input type=\"text\" name=\"q\" tabindex=\"1\" autocomplete=\"off\" id=\"search_form_input\" class=\"search__input search__input--adv js-search-input\" value=\"Fluence Labs\"><input id=\"search_form_input_clear\" class=\"search__clear  js-search-clear\" type=\"button\" tabindex=\"3\" value=\"X\"/><input id=\"search_button\" class=\"search__button  js-search-button\" type=\"submit\" tabindex=\"2\" value=\"S\" /><a id=\"search_dropdown\" class=\"search__dropdown\" href=\"javascript:;\" tabindex=\"4\"></a><div id=\"search_elements_hidden\" class=\"search__hidden  js-search-hidden\"></div></form></div></div><div class=\"zcm-wrap-wrap\"><div id=\"duckbar\" class=\"zcm-wrap  zcm-wrap--header  is-noscript-hidden\"></div><div class=\"zcm--right js-zcm-right\"></div></div></div><div class=\"header--aside js-header-aside\"></div></div><div id=\"zero_click_wrapper\" class=\"zci-wrap\"></div><div id=\"vertical_wrapper\" class=\"verticals\"></div><div id=\"web_content_wrapper\" class=\"content-wrap \"><div class=\"serp__top-right  js-serp-top-right\"></div><div class=\"serp__bottom-right  js-serp-bottom-right\"><div class=\"js-feedback-btn-wrap\"></div></div><div class=\"cw\"><div id=\"links_wrapper\" class=\"serp__results js-serp-results\"><div class=\"results--main\"><div class=\"search-filters-wrap\"><div class=\"js-search-filters search-filters\"></div></div><noscript><meta http-equiv=\"refresh\" content=\"0;URL=/html?q=Fluence%20Labs\"><link href=\"/css/noscript.css\" rel=\"stylesheet\" type=\"text/css\"><div class=\"msg msg--noscript\"><p class=\"msg-title--noscript\">You are being redirected to the non-JavaScript site.</p>Click <a href=\"/html/?q=Fluence%20Labs\">here</a> if it doesn't happen automatically.</div></noscript><div id=\"message\" class=\"results--message\"></div><div class=\"ia-modules js-ia-modules\"></div><div id=\"ads\" class=\"results--ads results--ads--main is-invisible js-results-ads\"></div><div id=\"links\" class=\"results is-invisible js-results\"></div></div><div class=\"results--sidebar js-results-sidebar\"><div class=\"sidebar-modules js-sidebar-modules\"></div><div class=\"is-invisible js-sidebar-ads\"></div></div></div></div></div><div id=\"bottom_spacing2\"> </div></div><script type=\"text/javascript\"></script><script type=\"text/JavaScript\">function nrji() {nrj('/t.js?q=Fluence%20Labs&l=us-en&s=0&dl=en&ct=RU&ss_mkt=us&p_ent=&ex=-1');DDG.deep.initialize('/d.js?q=Fluence%20Labs&l=us-en&s=0&dl=en&ct=RU&ss_mkt=us&vqd=3-167080984343175066141708440165596583146-24182048638364464238089820915205130627&p_ent=&ex=-1&sp=1');;};DDG.ready(nrji, 1);</script><script src=\"/g2662.js\"></script><script type=\"text/javascript\">DDG.page = new DDG.Pages.SERP({ showSafeSearch: 0, instantAnswerAds: false, hostRegion: \"euw\" });</script><div id=\"z2\"> </div><div id=\"z\"></div></body></html><script type=\"text/JavaScript\">DDG.index = DDG.index || {}; DDG.index.signalSummary = \"\";</script>")
 elapsed time: 356.953459ms

3> 
```
