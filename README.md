import asyncio
import random

async def process_user_earning(self, token: str, username: str, use_proxy: bool):
    while True:
        proxy = self.get_next_proxy_for_account(token) if use_proxy else None
        tasks = [
            self.user_today_earning(token, username, proxy),
            self.user_season_earning(token, username, proxy)
        ]
        today_earning, season_earning = await asyncio.gather(*tasks)
        
        if today_earning and season_earning:
            today_point = today_earning['data']
            season_point = season_earning['data']
            
            self.print_message(username, proxy, Fore.WHITE, 
                f"Earning Today {today_point} PTS"
                f"{Fore.MAGENTA + Style.BRIGHT} - {Style.RESET_ALL}"
                f"{Fore.WHITE + Style.BRIGHT}Earning Season {season_point} PTS{Style.RESET_ALL}"
            )
        
        await asyncio.sleep(random.randint(300, 600))  # Random delay 5-10 minutes

async def process_complete_mission(self, token: str, username: str, use_proxy: bool):
    while True:
        proxy = self.get_next_proxy_for_account(token) if use_proxy else None
        tasks = [self.social_media_tasks(token, username, task, proxy) for task in ["follow-x", "follow-telegram"]]
        await asyncio.gather(*tasks)
        
        ambassador_tasks = await self.ambassador_tasks(token, proxy)
        if ambassador_tasks:
            submit_tasks = [self.submit_tasks(token, task['_id'], proxy) for task in ambassador_tasks if task['status'] == 'UNCOMPLETED']
            await asyncio.gather(*submit_tasks)
        
        await asyncio.sleep(random.randint(600, 1200))  # Random delay 10-20 minutes

async def get_node_earning(self, token: str, username: str, nodes, use_proxy: bool):
    proxy = self.get_next_proxy_for_account(token) if use_proxy else None
    tasks = [self.single_node_data(token, username, node['_id'], proxy) for node in nodes]
    results = await asyncio.gather(*tasks)
    
    for node, result in zip(nodes, results):
        if result:
            today_earn = result['todayEarn']
            season_earn = result['seasonEarn']
            
            self.print_message(username, proxy, Fore.WHITE, 
                f"Node ID {node['node_id']}"
                f"{Fore.MAGENTA + Style.BRIGHT} - {Style.RESET_ALL}"
                f"{Fore.CYAN + Style.BRIGHT}Earning:{Style.RESET_ALL}"
                f"{Fore.WHITE + Style.BRIGHT} Today {today_earn} PTS {Style.RESET_ALL}"
                f"{Fore.MAGENTA + Style.BRIGHT}-{Style.RESET_ALL}"
                f"{Fore.WHITE + Style.BRIGHT} Season {season_earn} PTS {Style.RESET_ALL}"
            )

async def process_send_ping(self, token: str, username: str, use_proxy: bool):
    nodes = await self.process_loads_node_data(token, username, use_proxy)
    if nodes:
        tasks = []
        for node in nodes:
            if "id" in node:
                tasks.append(self.get_node_earning(token, username, [node], use_proxy))
            tasks.append(self.connect_websocket(token, username, node['node_id'], use_proxy, None))
        await asyncio.gather(*tasks)
