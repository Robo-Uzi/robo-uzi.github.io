---
layout: post
title:  "Get off the ISS"
date:   2026-03-09 17:43:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /upCTF-2026-Get-off-the-ISS/
---

**Title**: Get off the ISS

**Category**: misc

**Author**: abreu22

**Description**: The nerve is out of this world... [https://upctf26.tiagoabreu.pt/handout.zip](https://upctf26.tiagoabreu.pt/handout.zip) Validation endpoint: `http://46.225.117.62:30000/`

I get the challenge files:
```shell
unzip handout.zip  
Archive:  handout.zip    
  inflating: handout/station_b.sigmf-data  
  inflating: __MACOSX/handout/._station_b.sigmf-data  
  inflating: handout/station_c.sigmf-meta  
  inflating: __MACOSX/handout/._station_c.sigmf-meta  
  inflating: handout/challenge.md  
  inflating: __MACOSX/handout/._challenge.md  
  inflating: handout/station_a.sigmf-meta  
  inflating: __MACOSX/handout/._station_a.sigmf-meta  
  inflating: handout/station_c.sigmf-data  
  inflating: __MACOSX/handout/._station_c.sigmf-data  
  inflating: handout/station_b.sigmf-meta  
  inflating: __MACOSX/handout/._station_b.sigmf-meta  
  inflating: handout/station_a.sigmf-data  
  inflating: __MACOSX/handout/._station_a.sigmf-data
  
file *  
challenge.md:              Unicode text, UTF-8 text
station_a.sigmf-data:      data  
station_a.sigmf-meta:      JSON text data  
station_b.sigmf-data:      data  
station_b.sigmf-meta:      JSON text data  
station_c.sigmf-data:      data  
station_c.sigmf-meta:      JSON text data

cat station_a.sigmf-meta
{
    "global": {
        "core:datatype": "cf32_le",
        "core:sample_rate": 500000,
        "core:version": "1.0.0",
        "core:description": "IQ recording for Station A",
        "core:author": "Generated for CTF",
        "core:frequency": 146000000,
        "core:num_channels": 1,
        "core:sha512": "0de51da0429f9fe4fc6616ef5c069ba1e1ee9b5993c598eb6aa089a1020126da48601056e9353bdacbd6f6b233d6c42522b9fd8222023541a176d886ebb0fe28"
    },
    "captures": [
        {
            "core:sample_start": 0,
            "core:frequency": 146000000
        }
    ],
    "annotations": []
}

cat station_b.sigmf-meta
{
    "global": {
        "core:datatype": "cf32_le",
        "core:sample_rate": 500000,
        "core:version": "1.0.0",
        "core:description": "IQ recording for Station B",
        "core:author": "Generated for CTF",
        "core:frequency": 146000000,
        "core:num_channels": 1,
        "core:sha512": "45a37c2d30fb8d46abc96d609871c1c6eacf9eba10d846d2bebcf9e625ccffdc5bf2eb2f25360d6fef7d96b38f17f537390180f694d2412c7ef6fea1d57638d3"
    },
    "captures": [
        {
            "core:sample_start": 0,
            "core:frequency": 146000000
        }
    ],
    "annotations": []
}

cat station_c.sigmf-meta
{
    "global": {
        "core:datatype": "cf32_le",
        "core:sample_rate": 500000,
        "core:version": "1.0.0",
        "core:description": "IQ recording for Station C",
        "core:author": "Generated for CTF",
        "core:frequency": 146000000,
        "core:num_channels": 1,
        "core:sha512": "8a72004100795cd2b880a27eaa394123f62506ad5c5d26f81405c2c068ba866a5b8ea71505884937964ae0d170e3d0ae0401a039790461b83d5bd930f859ac94"
    },
    "captures": [
        {
            "core:sample_start": 0,
            "core:frequency": 146000000
        }
    ],
    "annotations": []
}

cat challenge.md  
I am trying to use the ISS radio repeater but some troll on my area is jamming the european uplink frequency...  
I've scattered some highly directional rotating antennas around the city to capture the radio sepctrum (0° North at t=0)  
  
Help me find this guy...  
  
To get the flag, select the jammer location in the validition endpoint.  
  
  
  
## Ground Station Coordinates  
station_a = (41.16791628282458, -8.688654341122007)  
station_b = (41.14456438258019, -8.675380772847733)  
station_c = (41.1413904156136, -8.609071069291119)
```

I wasnt sure what I was dealing with here at first. I opened one of the data files with `inspectrum` like this: `inspectrum station_a.sigmf-data` and I saw 3 distinct signals happening from each data file.

So each ground station recorded raw radio samples (IQ data) while its directional antenna rotated 360 degrees. A directional antenna receives signals strongest when it is pointing directly at the transmitter.

Because the antennas are rotating signal strength rises when the antenna points toward the jammer and signal strength falls when the antenna points away.

This produces a periodic peak in signal power every time the antenna completes a rotation. So from each recording we can determine:
1. How many rotations occurred
2. At what point in the rotation the signal was strongest

Since the `challenge.md` says: `0° North at t=0` we know the antenna orientation as a function of time.

Therefore:
time of peak signal will reveal antenna direction and the bearing to jammer. Each station gives one direction. With three stations you can triangulate the transmitter location.

Solve script:
```python
#!/usr/bin/env python3

import numpy as np
import json
import os
from scipy.signal import find_peaks, savgol_filter
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt

STATIONS = {
    'a': (41.16791628282458,  -8.688654341122007),
    'b': (41.14456438258019,  -8.675380772847733),
    'c': (41.1413904156136,   -8.609071069291119),
}

DATA_DIR = '.'

def load_sigmf(station):
    meta_path = os.path.join(DATA_DIR, f'station_{station}.sigmf-meta')
    data_path = os.path.join(DATA_DIR, f'station_{station}.sigmf-data')
    with open(meta_path) as f:
        meta = json.load(f)
    sample_rate = meta['global']['core:sample_rate']
    raw = np.fromfile(data_path, dtype='<f4')
    iq  = raw[0::2] + 1j * raw[1::2]
    return iq, sample_rate, meta

def compute_power_envelope(iq, chunk_size=5000):
    n_chunks = len(iq) // chunk_size
    iq_trunc = iq[:n_chunks * chunk_size].reshape(n_chunks, chunk_size)
    power = np.abs(iq_trunc).mean(axis=1)
    return power

def detect_n_rotations(smooth):
    autocorr = np.correlate(smooth - smooth.mean(), smooth - smooth.mean(), mode='full')
    autocorr = autocorr[len(autocorr)//2:]
    autocorr /= autocorr[0]
    peaks_ac, _ = find_peaks(autocorr[10:], height=0.1)
    if len(peaks_ac) > 0:
        period_chunks = peaks_ac[0] + 10
        return len(smooth) / period_chunks, period_chunks
    return 1, len(smooth)

def find_bearing(power, n_rotations):
    # use modulo to find peak position within one rotation cycle. The antenna sweeps 360 degrees per rotation. We need to know at what angle within a single sweep the peak occurred. Not its position in the entire recording.
    chunks_per_rotation = len(power) / n_rotations

    # Find global peak (highest amplitude across all rotations)
    peak_idx = np.argmax(power)

    peak_in_cycle = peak_idx % chunks_per_rotation
    fraction_in_cycle = peak_in_cycle / chunks_per_rotation
    bearing = fraction_in_cycle * 360.0

    return bearing, peak_idx, peak_in_cycle, fraction_in_cycle

def gc_bearing(lat1, lon1, lat2, lon2):
    lat1, lon1, lat2, lon2 = map(np.radians, [lat1, lon1, lat2, lon2])
    dlon = lon2 - lon1
    x = np.sin(dlon) * np.cos(lat2)
    y = np.cos(lat1)*np.sin(lat2) - np.sin(lat1)*np.cos(lat2)*np.cos(dlon)
    return (np.degrees(np.arctan2(x, y)) + 360) % 360

def destination(lat, lon, bearing_deg, distance_m):
    R = 6_371_000
    d = distance_m / R
    b = np.radians(bearing_deg)
    lat1, lon1 = np.radians(lat), np.radians(lon)
    lat2 = np.arcsin(np.sin(lat1)*np.cos(d) + np.cos(lat1)*np.sin(d)*np.cos(b))
    lon2 = lon1 + np.arctan2(np.sin(b)*np.sin(d)*np.cos(lat1),
                              np.cos(d) - np.sin(lat1)*np.sin(lat2))
    return np.degrees(lat2), np.degrees(lon2)

def ray_intersect_2d(lat1, lon1, az1, lat2, lon2, az2):
    def az_to_vec(az):
        r = np.radians(az)
        return np.array([np.sin(r), np.cos(r)])
    d1 = az_to_vec(az1)
    d2 = az_to_vec(az2)
    def to_meters(lat, lon):
        dlat = (lat - lat1) * 111_320
        dlon = (lon - lon1) * 111_320 * np.cos(np.radians(lat1))
        return np.array([dlon, dlat])
    p1 = np.array([0.0, 0.0])
    p2 = to_meters(lat2, lon2)
    A = np.column_stack([d1, -d2])
    if abs(np.linalg.det(A)) < 1e-10:
        return None
    t, s = np.linalg.solve(A, p2 - p1)
    intersection_m = p1 + t * d1
    lat_j = lat1 + intersection_m[1] / 111_320
    lon_j = lon1 + intersection_m[0] / (111_320 * np.cos(np.radians(lat1)))
    return lat_j, lon_j

def triangulate_ls(stations_coords, bearings):
    keys = list(stations_coords.keys())
    results = []
    for i in range(len(keys)):
        for j in range(i+1, len(keys)):
            s1, s2 = keys[i], keys[j]
            lat1, lon1 = stations_coords[s1]
            lat2, lon2 = stations_coords[s2]
            pt = ray_intersect_2d(lat1, lon1, bearings[s1],
                                  lat2, lon2, bearings[s2])
            if pt:
                results.append(pt)
                print(f"  Pair {s1}-{s2}: intersection = {pt[0]:.6f}, {pt[1]:.6f}")
    if not results:
        return None
    return np.mean([r[0] for r in results]), np.mean([r[1] for r in results])

def main():
    bearings = {}
    fig, axes = plt.subplots(1, 3, figsize=(18, 4))

    for idx, station in enumerate('abc'):
        print(f"\n[Station {station.upper()}] Loading...")
        iq, sr, meta = load_sigmf(station)
        total_samples = len(iq)
        duration_s = total_samples / sr
        print(f"  Samples: {total_samples:,}  |  Duration: {duration_s:.1f}s")

        power = compute_power_envelope(iq, chunk_size=5000)
        smooth = savgol_filter(power, window_length=min(51, len(power)//10*2+1), polyorder=3)

        n_rotations, period_chunks = detect_n_rotations(smooth)
        n_rotations = round(n_rotations)  # snap to integer
        print(f"  Rotations: {n_rotations}  |  Chunks/rotation: {period_chunks:.1f}")

        bearing, peak_idx, peak_in_cycle, fraction = find_bearing(smooth, n_rotations)
        bearings[station] = bearing
        print(f"  Global peak chunk: {peak_idx}  |  In-cycle chunk: {peak_in_cycle:.1f}  |  Fraction: {fraction:.4f}  |  Bearing: {bearing:.2f}°")

        # Plot one rotation's worth of data to show the peak clearly
        cpr = int(len(smooth) / n_rotations)
        # which rotation holds the peak?
        rot_num = peak_idx // cpr
        seg = smooth[rot_num*cpr : (rot_num+1)*cpr]
        t_seg = np.linspace(rot_num * (duration_s/n_rotations),
                            (rot_num+1) * (duration_s/n_rotations), len(seg))
        t_all = np.linspace(0, duration_s, len(smooth))
        axes[idx].plot(t_all, smooth, color='lightblue', label='All rotations')
        axes[idx].plot(t_seg, seg, color='blue', label=f'Peak rotation #{rot_num+1}')
        axes[idx].axvline(t_all[peak_idx], color='red', linestyle='--',
                          label=f'Peak → {bearing:.1f}°')
        axes[idx].set_title(f'Station {station.upper()} — Bearing: {bearing:.1f}°')
        axes[idx].set_xlabel('Time (s)')
        axes[idx].set_ylabel('Amplitude')
        axes[idx].legend(fontsize=8)

    plt.tight_layout()
    plt.savefig('power_envelopes.png', dpi=120)
    print("\nSaved power_envelopes.png")

    print(f"\nBearings (from North, clockwise):")
    for s, b in bearings.items():
        print(f"    Station {s.upper()}: {b:.2f}°")

    print("\nTriangulating...")
    jammer = triangulate_ls(STATIONS, bearings)

    if jammer:
        print(f"  Jammer location:")
        print(f"  Latitude:  {jammer[0]:.6f}")
        print(f"  Longitude: {jammer[1]:.6f}")
        print(f"  Google Maps: https://maps.google.com/?q={jammer[0]:.6f},{jammer[1]:.6f}")

        print("\nVerification (bearing errors should now be small):")
        for s, (lat, lon) in STATIONS.items():
            expected = gc_bearing(lat, lon, jammer[0], jammer[1])
            measured = bearings[s]
            diff = abs(expected - measured)
            if diff > 180: diff = 360 - diff
            print(f"    Station {s.upper()}: measured={measured:.2f}°  expected={expected:.2f}°  diff={diff:.2f}°")

        # Map
        fig2, ax2 = plt.subplots(figsize=(8, 8))
        colors = {'a': 'blue', 'b': 'green', 'c': 'orange'}
        for s, (lat, lon) in STATIONS.items():
            ax2.plot(lon, lat, 'o', color=colors[s], markersize=10, label=f'Station {s.upper()}')
            end_lat, end_lon = destination(lat, lon, bearings[s], 8000)
            ax2.plot([lon, end_lon], [lat, end_lat], '--', color=colors[s], alpha=0.7)
        ax2.plot(jammer[1], jammer[0], 'r*', markersize=22, label='JAMMER', zorder=5)
        ax2.set_title('Jammer Triangulation')
        ax2.set_xlabel('Longitude')
        ax2.set_ylabel('Latitude')
        ax2.legend()
        ax2.grid(True)
        plt.tight_layout()
        plt.savefig('triangulation.png', dpi=120)
        print("Saved triangulation.png")

if __name__ == '__main__':
    main()
```

Output:
```shell
python3 coords.py  
  
[Station A] Loading...  
Samples: 15,000,000  |  Duration: 30.0s  
Rotations: 3  |  Chunks/rotation: 1000.0  
Global peak chunk: 1283  |  In-cycle chunk: 283.0  |  Fraction: 0.2830  |  Bearing: 101.88°  
  
[Station B] Loading...  
Samples: 15,000,000  |  Duration: 30.0s  
Rotations: 3  |  Chunks/rotation: 1000.0  
Global peak chunk: 181  |  In-cycle chunk: 181.0  |  Fraction: 0.1810  |  Bearing: 65.16°  
  
[Station C] Loading...  
Samples: 15,000,000  |  Duration: 30.0s  
Rotations: 3  |  Chunks/rotation: 1000.0  
Global peak chunk: 2882  |  In-cycle chunk: 882.0  |  Fraction: 0.8820  |  Bearing: 317.52°  
  
Saved power_envelopes.png  
  
Bearings (from North, clockwise):  
Station A: 101.88°  
Station B: 65.16°  
Station C: 317.52°  
  
Triangulating...  
Pair a-b: intersection = 41.159175, -8.633454  
Pair a-c: intersection = 41.158635, -8.630048  
Pair b-c: intersection = 41.159854, -8.631522  
Jammer location:  
Latitude:  41.159221  
Longitude: -8.631675  
Google Maps: https://maps.google.com/?q=41.159221,-8.631675  
  
Verification (bearing errors should now be small):  
Station A: measured=101.88°  expected=101.44°  diff=0.44°  
Station B: measured=65.16°  expected=65.98°  diff=0.82°  
Station C: measured=317.52°  expected=316.34°  diff=1.18°  
Saved triangulation.png
```

`triangulation.png` shows the correct triangulation:

![Alt text](/images/triangulation.png)

I go to `https://maps.google.com/?q=41.159221,-8.631675` and find the location of the jammer (around `Av. da Boavista 604-610 Piso 0, 4149-071 Porto, Portugal`).

I click on this location on the map on the validation endpoint and receive the flag!

`upCTF{fl4t_e4rth3rs_cou1d_n3v3r-HuJiIHkQbc7f7dff}`