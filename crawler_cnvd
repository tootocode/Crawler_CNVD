# coding:utf-8
import requests
from lxml import etree
import xlsxwriter
from requests.utils import add_dict_to_cookiejar
import execjs
import hashlib
import json
import re
import time
import datetime


f = open(r"C:\Users\DELL\Desktop\爬1.txt", "w")

def get__jsl_clearance_s(data):
    chars = len(data['chars'])
    for i in range(chars):
        for j in range(chars):
            __jsl_clearance_s = data['bts'][0] + data['chars'][i] + data['chars'][j] + data['bts'][1]
            encrypt = None
            if data['ha'] == 'md5':
                encrypt = hashlib.md5()
            elif data['ha'] == 'sha1':
                encrypt = hashlib.sha1()
            elif data['ha'] == 'sha256':
                encrypt = hashlib.sha256()
            encrypt.update(__jsl_clearance_s.encode())
            result = encrypt.hexdigest()
            if result == data['ct']:
                return __jsl_clearance_s


def setCookie(url):
    global session
    session = requests.session()

    response1 = session.get(url)
    jsl_clearance_s = re.findall(r'cookie=(.*?);location', response1.text)[0]
    jsl_clearance_s = str(execjs.eval(jsl_clearance_s)).split('=')[1].split(';')[0]
    add_dict_to_cookiejar(session.cookies, {'__jsl_clearance_s': jsl_clearance_s})

    response2 = session.get(url)
    data = json.loads(re.findall(r';go\((.*?)\)', response2.text)[0])
    jsl_clearance_s = get__jsl_clearance_s(data)
    add_dict_to_cookiejar(session.cookies, {'__jsl_clearance_s': jsl_clearance_s})


class CNVD(object):
    def __init__(self, type_id):
        self.type_id = str(type_id)
        self.host_url = "https://www.cnvd.org.cn"
        # self.start_url = "https://www.cnvd.org.cn/flaw/typeResult?typeId=" + self.type_id
        # self.base_url = "https://www.cnvd.org.cn/flaw/typeResult?typeId={}&max={}&offset={}"


    '''def get_max_page(self):
        """
        通过列表页第一页获取最大分页页码, 最多选取10000条记录
        Returns: [type] -- [min(最大分页页码, 500)，整数]
        """
        response = session.get(url=self.start_url)
        content = response.text
        html = etree.HTML(content)
        max_page = html.xpath("/html/body/div/span[3]/text()")[0]
        max_page = re.findall(r"\d+", max_page)
        print("页数", max_page[0])
        if max_page:
            max_page = int(max_page[0])
            # return min(max_page, 5)
            return max_page'''

    def get_max_page(self):
        """
        通过列表页第一页获取最大分页页码, 最多选取10000条记录
        Returns: [type] -- [min(最大分页页码, 500)，整数]
        """
        # response = session.get(url=self.start_url)
        max_page = 0
        response = session.post(url=r"https://www.cnvd.org.cn/flaw/list.htm?flag=true",
                                data={'keyword': self.type_id, 'condition': 0})

        # response.keep_alive = False
        i = 1
        while (response.status_code != 200 and i <= 50):
            i = i + 1
            response.close()
            time.sleep(2)
            response = session.post(url=r"https://www.cnvd.org.cn/flaw/list.htm?flag=true",
                                data={'keyword': self.type_id, 'condition': 0})
            # content = response.text
            # print("zhuye页请求失败", response.status_code)
        response.close()

        content = response.text
        html = etree.HTML(content)
        try:
            max_page = html.xpath("//div[@class='pages clearfix']//a[last()-1]/text()")[0]
        except BaseException:
            max_page = 1
        print(max_page)
        if max_page:
            max_page = int(max_page)
            return max_page

    def get_list_page(self, max_page):
        """
        获取列表页html内容
        Arguments: max_page {[int]} -- [最大分页页码]
        """
        max = 10
        flag = 0;
        for page in range(max_page):
            flag = flag + 1
            print("正在爬取列表页第<%s>页" % str(page))
            offset = page * max
            print(page, offset, max_page)
            url = r'https://www.cnvd.org.cn/flaw/list.htm?flag=true'
            # url = "https://www.cnvd.org.cn/flaw/typeResult?typeId={"+str(27)+"}&max={"+str(20)+"}&offset={"+offset+"}"
            print(url)
            time.sleep(2)
            respons = session.post(url=url, data={'keyword': self.type_id, 'max': max, 'offset': offset, 'numPerPage': 10,
                                                  'condition': 0})
            content = respons.text
            i = 1
            while (respons.status_code != 200 and i <= 50):
                i = i + 1
                respons.close()
                time.sleep(2)
                respons = session.post(url=url, data={'keyword': self.type_id, 'max': max, 'offset': offset, 'numPerPage': 10,
                                                      'condition': 0})
                content = respons.text
                # print("zhuye页请求失败", response.status_code)
            respons.close()
            # print("正在爬取列表页第<%s>页" % str(page), response.status_code)
            yield content

    def parse_list_page(self, content):
        """
        获取列表页中的详情页href，返回href列表
        Arguments:
            content {[str]} -- [列表页html内容]
        Returns:
            [list] -- [详情页href列表]
        """
        html = etree.HTML(content)
        href_list = html.xpath("//tbody/tr/td//a/@href")
        print(href_list)
        return href_list

    def handle_str(self, td_list):
        """
        字符串去除空格\r\n\t等字符
        Arguments:
            td_list {[list]} -- [获取文本的列表]
        Returns:
            [type] -- [返回处理后的合并文本]
        """
        result = ''
        for td_str in td_list[1:]:
            result += td_str.strip()
        result = "".join(result.split())
        # 去除"危害级别"中的括号
        if result.endswith("()"):
            result = result[:-2]
        return result

    def parse_detail_page(self, content):
        """
        从详情页中提取信息
        Arguments:
            content {[str]} -- [详情页html内容]
        Returns:
            [dict] -- [提取信息字典]
        """
        try:
            html = etree.HTML(content)
            item = {}
            item['漏洞名称'] = '' if len(
                html.xpath("//div[@class='blkContainer']//div[@class='blkContainerSblk']/h1/text()")) == 0 else \
                html.xpath("//div[@class='blkContainer']//div[@class='blkContainerSblk']/h1/text()")[0]
            tr_list = html.xpath("//div[@class='blkContainer']//div[@class='blkContainerSblk']//tbody/tr")
            # print(tr_list)
            for tr in tr_list:
                td_list = tr.xpath("./td/text()|./td/a/text()")
                print(td_list)
                if len(td_list) >= 2:
                    item[td_list[0]] = self.handle_str(td_list)
            print(item)
            f.write(str(item) + "\n")
            f.flush()
        except BaseException:
            print("item 失败")
        '''CNVD = html.xpath("/html/body/div[4]/div[1]/div[1]/div[1]/div[2]/div[1]/table/tbody/tr[1]/td[2]/text()")
        CNVD = ''.join(CNVD)
        Hazardlevel = html.xpath("/html/body/div[4]/div[1]/div[1]/div[1]/div[2]/div[1]/table/tbody/tr[3]/td[2]/text()")
        Hazardlevel = ''.join(Hazardlevel)
        print(CNVD.strip(), " ", Hazardlevel.strip())'''

    def get_detail_info(self, href):
        """
        通过详情页href，获取详情页html内容
        Arguments:
            href {[str]} -- [详情页href]
        Returns:
            [调用详情页解析函数] -- [提取信息]
        """
        # host_url = "https://www.cnvd.org.cn" <a href="/flaw/show/CNVD-2021-02038" title="MiniCMS目录遍历漏洞（CNVD-2021-02038）">
        url = self.host_url + href
        print("正在爬取详情页<%s>" % url)
        # print("正在爬取详情页<%s>" % url)
        respons = session.get(url=url)
        time.sleep(2)
        i = 0
        while (respons.status_code != 200 and i <= 20):
            i = i + 1
            respons.close()
            time.sleep(2)
            respons = session.get(url=url)
            # content = response.text
            # print("详细页请求失败", respons.status_code)
        respons.close()
        return self.parse_detail_page(respons.content.decode("utf-8"))


def vulnerability_xlsx(vlist, type):
    '''worksheet = workbook.add_worksheet(type)

    column_names = ['漏洞名称', 'CNVD-ID', '危害级别', 'CVE ID', '漏洞类型', '影响产品', '漏洞描述']
    for i in range(len(column_names)):
        worksheet.write(0, i, column_names[i])

    for i in range(len(vlist)):
        for j in range(len(column_names)):
            worksheet.write(i + 1, j, vlist[i][column_names[j]] if column_names[j] in vlist[i].keys() else '')'''


def spider(type_id):
    vlist = []
    cnvd = CNVD(type_id)
    max_page = cnvd.get_max_page()
    content_generator = cnvd.get_list_page(max_page)
    for content in content_generator:
        href_list = cnvd.parse_list_page(content)
        for href in href_list:
            item = cnvd.get_detail_info(href)
            vlist.append(item)


if __name__ == "__main__":

    list = [
            'ASUS',
            'BUFFALO',
            'MikroTik',
            'Netgear',
            'NetGear',
            'NetVanta',
            'Cisco',
            'Digi',
            'DLink',
            'DrayTek',
            'Corecess',
            'UBNT',
            'Eltex',
            'ZyXEL',
            'FiberHome',
            'Foundry',
            'Freecomm',
            'Yamaha',
            'Netopia',
            '迈普',
            '腾达',
            'IBM',
            'Infinet',
            'Juniper',
            'MAX',
            'Meraki',
            'MOXA',
            'Mikrotik',
            'Red',
            '锐捷',
            'Asus',
            'WL',
            'Binatone',
            'Evo',
            'FAST',
            'Fast',
            'Mercury',
            'MENN',
            'PACIFIC',
            'Rosewill',
            'SCLD',
            'BEC',
            'PingCom',
            'Advantech',
            ]  # RT,CG
    begin = datetime.datetime.now()
    for i in range(len(list)):
        print(list[i])
        setCookie('https://www.cnvd.org.cn/')
        spider(list[i])
    time.sleep(2)
    end = datetime.datetime.now()
    total_time = end - begin
    print('爬取漏洞的时间: ', total_time, sep='')
