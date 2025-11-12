#!/usr/bin/env python3
# File: quick_grid.py
# Create a simple contact sheet (grid) from a folder of images.
# Usage: python quick_grid.py --dir ./images --cols 3 --cell 540 --gap 12 --bg "#111827" --out grid.jpg

import argparse, glob, os, math
from PIL import Image, ImageOps, ImageColor

def load_images(folder):
    exts = ("*.jpg","*.jpeg","*.png","*.webp","*.JPG","*.PNG","*.JPEG","*.WEBP")
    files = []
    for e in exts: files += sorted(glob.glob(os.path.join(folder, e)))
    return files

def square_cover(img, size):
    w,h = img.size
    if w==h: return img.resize((size,size), Image.LANCZOS)
    if w>h:  x=(w-h)//2; img=img.crop((x,0,x+h,h))
    else:    y=(h-w)//2; img=img.crop((0,y,w,y+w))
    return img.resize((size,size), Image.LANCZOS)

def hex_rgb(h): return ImageColor.getrgb(h)

def make_grid(files, cols, cell, gap, bg_hex):
    n=len(files); rows=math.ceil(n/cols)
    W=cols*cell+(cols+1)*gap; H=rows*cell+(rows+1)*gap
    canvas=Image.new("RGB",(W,H),hex_rgb(bg_hex))
    for i,f in enumerate(files):
        im=Image.open(f).convert("RGB")
        im=square_cover(im, cell)
        r=i//cols; c=i%cols
        x=gap+c*(cell+gap); y=gap+r*(cell+gap)
        canvas.paste(im,(x,y))
    return canvas

def main():
    p=argparse.ArgumentParser(description="Quick image grid/contact sheet.")
    p.add_argument("--dir", required=True, help="Folder of images")
    p.add_argument("--cols", type=int, default=3)
    p.add_argument("--cell", type=int, default=540, help="Square cell size (px)")
    p.add_argument("--gap", type=int, default=12)
    p.add_argument("--bg", default="#111827")
    p.add_argument("--out", default="grid.jpg")
    args=p.parse_args()

    files=load_images(args.dir)
    if not files: raise SystemExit("No images found.")
    grid=make_grid(files, args.cols, args.cell, args.gap, args.bg)
    grid.save(args.out, quality=95, optimize=True, progressive=True, subsampling="4:2:0")
    print(f"✅ Saved {args.out} ({len(files)} images, {args.cols}×{math.ceil(len(files)/args.cols)} grid)")

if __name__=="__main__":
    main()

