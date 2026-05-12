---
layout: post
title:  "Panic In the Northern Quadrant (part 1/3)"
date:   2026-05-12 19:18:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THCON-CTF-2026-panic-in-the-northern-quadrant/
---

**Category**: Web

**Author**: [chepycou](https://www.ar-lacroix.fr/)

**Description**: We have found a legacy server for the SST facility that's behind Santo Domingo. It does not seem to have been compromised yet, but all employees lost access to their Single Sign On and the facility is physically unreachable. We first need you to recon the facility's website which from what we can remember the last time we audited them is in a pitiful security state with a bunch of critical vulnerabilities like file uploads and SSRFs. They have probably patched them since, but if they have not, we should find ways of regaining control.

Keep us posted whenever you find something interesting. Getting hold of this facility may help us add more friendly robots to our forces !

[http://panic-in-the-northern-quadrant.ctf.thcon.party:8080](http://panic-in-the-northern-quadrant.ctf.thcon.party:8080)

I go to the site:

![Alt text](/images/thconweb2.png)

I started with looking at the source code. I find some useful comments:
```html
<!-- <a href="register.ph">Create an account</a> -->
<!-- TODO : Fix null-byte file upload vulnerability -->
<!-- build:app --> 
<script>
... etc javascript etc ...
</script> 
<!-- end:app -->
```

Most of the functionality on the page seems broken and leads to a 404. 

I beautify the js on the page:
```javascript
var _0x4a1f = [
		'api/v1/units',
		'api/v1/units/status',
		'api/v2/fleet',
		'api/v2/fleet/sync',
		'download-legacy',
		'backup',
		'api/v3/telemetry',
		'api/v3/telemetry/stream',
		'api/internal/auth',
		'api/internal/logs',
		'api/internal/config',
		'api/internal/export',
		'db/primary',
		'db/replica',
		'db/archive',
		'db/metrics',
		'sys/health',
		'sys/version',
		'sys/reboot',
		'sys/diagnostics'
	], ROUTES = {
		unitsV1: _0x4a1f[0],
		unitsStatus: _0x4a1f[1],
		fleetV2: _0x4a1f[2],
		fleetSync: _0x4a1f[3],
		legacy: _0x4a1f[4],
		backup: _0x4a1f[5],
		telemetry: _0x4a1f[6],
		telemetryStream: _0x4a1f[7],
		authInternal: _0x4a1f[8],
		logsInternal: _0x4a1f[9],
		configInternal: _0x4a1f[10],
		exportInternal: _0x4a1f[11],
		dbPrimary: _0x4a1f[12],
		dbReplica: _0x4a1f[13],
		dbArchive: _0x4a1f[14],
		dbMetrics: _0x4a1f[15],
		sysHealth: _0x4a1f[16],
		sysVersion: _0x4a1f[17],
		sysReboot: _0x4a1f[18],
		sysDiag: _0x4a1f[19]
	};
/*function backup(){_fetch(ROUTES.backup, {"headers" : {"Content-Type": "application/x-www-form-urlencoded"}, "method":"POST", "body" : atob("dXNlcm5hbWU9c3N0JnBhc3N3b3JkPVRIQ3tzM2N1cjNwNDU1fQ==")})}function a(){_fetch(R.u).then(t=>{if(!t)return;try{var d=JSON.parse(t);console.log("[Units] Loaded",d.length||0,"units");window.lastUnitsData=d}catch(e){console.warn("[Units] Invalid JSON")}}).catch(e=>console.error("[Units] Failed:",e))};function b(){_fetch(R.us).then(t=>{if(!t)return;try{var s=JSON.parse(t);console.log("[Units Status]",s);window.lastUnitsStatus=s}catch(e){console.warn("[Units Status] Parse failed")}}).catch(e=>console.error("[Units Status] Failed:",e))};function c(){_fetch(R.f).then(t=>{if(!t)return;try{var fl=JSON.parse(t);console.log("[Fleet] Data received",fl);window.lastFleetData=fl}catch(e){console.warn("[Fleet] Invalid format")}}).catch(e=>console.error("[Fleet] Failed:",e))};function d(){console.log("[Fleet Sync] Starting...");_fetch(R.fs,{method:"POST"}).then(t=>{console.log("[Fleet Sync] Success",t||"OK");window.lastSyncTime=Date.now()}).catch(e=>console.error("[Fleet Sync] Failed:",e))};function initFleetSystem(){console.log("[API] Initializing...");a();b();c();_poll(R.fs,3e4);console.log("[API] System ready");}*/
!function () {
	var _cfg = {
		retry: 3,
		timeout: 5000,
		apiBase: '/',
		debug: !1
	};
	function _fetch(r, o) {
		o = o || {};
		var u = _cfg.apiBase + r;
		return fetch(u, {
			method: o.method || 'GET',
			headers: o.headers || {},
			body: o.body || null
		}).then(function (r) {
			if (!r.ok)
				throw new Error('HTTP ' + r.status);
			return r.text();
		}).catch(function (r) {
			_cfg.debug && console.error('[SST]', r);
		});
	}
	function _poll(r, t) {
		setInterval(function () {
			_fetch(r).then(function (r) {
				_cfg.debug && console.log('[poll]', r);
			});
		}, t || 60000);
	}
	function _noop() {
		return null;
	}
	function _initTelemetry() {
		_poll(ROUTES.telemetry, 90000);
	}
	_noop(ROUTES.dbPrimary);
	_noop(ROUTES.dbReplica);
	_noop(ROUTES.dbArchive);
	_noop(ROUTES.dbMetrics);
	_noop(ROUTES.sysReboot);
	_noop(ROUTES.authInternal);
	_noop(ROUTES.configInternal);
	_noop(ROUTES.fleetSync);
	_noop(ROUTES.telemetryStream);
	_noop(ROUTES.logsInternal);
	_noop(ROUTES.exportInternal);
	_noop(ROUTES.sysVersion);
	_noop(ROUTES.sysDiag);
	function _initFleet() {
		_fetch(ROUTES.unitsV1).then(function (r) {
			_cfg.debug && console.log('[fleet]', r);
		});
	}
	function _initHealth() {
		_fetch(ROUTES.sysHealth).then(function (r) {
			_cfg.debug && console.log('[health]', r);
		});
	}
	_initTelemetry();
	_initFleet();
	_initHealth();
}();
```

This js is commented out:
```javascript
function backup() {
	_fetch(ROUTES.backup, {
		'headers': { 'Content-Type': 'application/x-www-form-urlencoded' },
		'method': 'POST',
		'body': atob('dXNlcm5hbWU9c3N0JnBhc3N3b3JkPVRIQ3tzM2N1cjNwNDU1fQ==')
	});
}
function a() {
	_fetch(R.u).then(t => {
		if (!t)
			return;
		try {
			var d = JSON.parse(t);
			console.log('[Units] Loaded', d.length || 0, 'units');
			window.lastUnitsData = d;
		} catch (e) {
			console.warn('[Units] Invalid JSON');
		}
	}).catch(e => console.error('[Units] Failed:', e));
}
;
function b() {
	_fetch(R.us).then(t => {
		if (!t)
			return;
		try {
			var s = JSON.parse(t);
			console.log('[Units Status]', s);
			window.lastUnitsStatus = s;
		} catch (e) {
			console.warn('[Units Status] Parse failed');
		}
	}).catch(e => console.error('[Units Status] Failed:', e));
}
;
function c() {
	_fetch(R.f).then(t => {
		if (!t)
			return;
		try {
			var fl = JSON.parse(t);
			console.log('[Fleet] Data received', fl);
			window.lastFleetData = fl;
		} catch (e) {
			console.warn('[Fleet] Invalid format');
		}
	}).catch(e => console.error('[Fleet] Failed:', e));
}
;
function d() {
	console.log('[Fleet Sync] Starting...');
	_fetch(R.fs, { method: 'POST' }).then(t => {
		console.log('[Fleet Sync] Success', t || 'OK');
		window.lastSyncTime = Date.now();
	}).catch(e => console.error('[Fleet Sync] Failed:', e));
}
;
function initFleetSystem() {
	console.log('[API] Initializing...');
	a();
	b();
	c();
	_poll(R.fs, 30000);
	console.log('[API] System ready');
}
```

Decode the base64:
```shell
echo "dXNlcm5hbWU9c3N0JnBhc3N3b3JkPVRIQ3tzM2N1cjNwNDU1fQ==" | base64 -d  
username=sst&password=THC{s3cur3p455}
```

`THC{s3cur3p455}`