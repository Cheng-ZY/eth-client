# 
# EthClient
PHP Eth RPC客户端
##
[![Latest Stable Version](https://poser.pugx.org/czy/eth-client/v)](https://packagist.org/packages/czy/eth-client)
[![Require](https://poser.pugx.org/czy/eth-client/require/ext-json)](https://packagist.org/packages/vufind/vufind)
[![Require](https://poser.pugx.org/czy/eth-client/require/ext-gmp)](https://packagist.org/packages/vufind/vufind)
[![Require](https://poser.pugx.org/czy/eth-client/require/ext-bcmath)](https://packagist.org/packages/vufind/vufind)
[![Require](https://poser.pugx.org/czy/eth-client/require/simplito/elliptic-php)](https://packagist.org/packages/simplito/elliptic-php)
[![Require](https://poser.pugx.org/czy/eth-client/require/kornrunner/keccak)](https://packagist.org/packages/kornrunner/keccak)
[![Require](https://poser.pugx.org/czy/eth-client/require/guzzlehttp/guzzle)](https://packagist.org/packages/guzzlehttp/guzzle)
[![Total Downloads](https://poser.pugx.org/czy/eth-client/downloads)](https://packagist.org/packages/czy/eth-client)
[![License](https://poser.pugx.org/czy/eth-client/license)](https://packagist.org/packages/czy/eth-client)
# 在此库的基础上做修改
https://github.com/myxtype/ethereum-client
## 依赖 php-gmp 扩展，linux-alpine-php8.1 环境添加命令
```shell
apk add gmp-dev
apk add php81-gmp
```
## 安装
```shell
composer require czy/eth-client
```
## 在 Hyperf-3(PHP8.1) 框架上使用
#### 更多方法搜索 'eth jsonrpc' ,EthClient 调用 'eth_' 开头的方法都为 eth jsonrpc 接口方法
```php
<?php

declare(strict_types=1);
/**
 * Czy
 */
namespace App\Controller;

use Czy\Ethereum\EthClient;
use Czy\Ethereum\Utils;
use Exception;
use GuzzleHttp\Client;
use Hyperf\HttpServer\Annotation\Controller;
use Hyperf\HttpServer\Annotation\RequestMapping;
use Psr\Http\Message\ResponseInterface;

#[Controller(prefix: 'Eth')]
class Eth extends AbstractController
{
    /**
     * @var EthClient eth 客户端
     */
    protected EthClient $ethClient;

    /**
     * @var string eth 节点, 这里为 eth-goerli 测试节点
     */
    private string $ethUri = 'https://eth-goerli.g.alchemy.com/v2/cQ_wTHz6237vKR8yagHHTyrv1XPug_Oj';

    public function __construct()
    {
        $client = new Client([
            'base_uri' => $this->ethUri,
            'timeout' => 10,
        ]);
        $this->ethClient = new EthClient($client);
    }

    /**
     * eth 协议版本。
     * @return ResponseInterface
     */
    #[RequestMapping(path: 'protocolVersion')]
    public function protocolVersion(): ResponseInterface
    {
        $result = $this->ethClient->eth_protocolVersion();
        return $this->responseJson([
            'code' => 200,
            'msg' => 'Eth/protocolVersion',
            'data' => hexdec($result),
        ]);
    }

    /**
     * 根据钱包地址获取钱包余额。
     */
    #[RequestMapping(path: 'getBalanceByAddress')]
    public function getBalanceByAddress(): ResponseInterface
    {
        // ethClient 调用 'eth_' 开头的方法都为 eth jsonrpc 接口方法
        $result = $this->ethClient->eth_getBalance('0xda2F2930FfA9eA1D52f066F02FF2C4bE796E2f15');
        // 转换单位 wei to eth
        $balance = Utils::weiToEth($result);
        return $this->responseJson([
            'code' => 200,
            'msg' => 'Eth/getBalanceByAddress',
            'data' => $balance,
        ]);
    }

    /**
     * eth 交易。
     * @throws Exception
     */
    #[RequestMapping(path: 'transaction')]
    public function transaction(): ResponseInterface
    {
        // 转出、转入钱包地址
        $from = '0xda2F2930FfA9eA1D52f066F02FF2C4bE796E2f15';
        $to = '0x0e9022479bd23b749cdE790e8c348C39401FE481';
        $value = '0.001'; // eth
        // from 钱包私钥
        $this->ethClient->addPrivateKeys(['a44a3081b4166fe75d8a54c23c9cba9c2f759176f4f2d824a284ce851bb56c9f']);
        // 交易数据
        $trans = [
            'from' => $from,
            'to' => $to,
            'value' => Utils::ethToWei($value, true),
            'data' => '0x',
        ];
        // gas 数量
        $trans['gas'] = dechex((int) hexdec($this->ethClient->eth_estimateGas($trans)));
        // gas 单价
        $trans['gasPrice'] = $this->ethClient->eth_gasPrice();
        // 交易序号
        $trans['nonce'] = $this->ethClient->eth_getTransactionCount($from, 'pending');

        // 发送交易请求，返回交易哈希
        $transHash = $this->ethClient->sendTransaction($trans);

        return $this->responseJson([
            'code' => 200,
            'msg' => 'Eth/Transaction',
            'data' => [
                'transHash' => $transHash,
                'transactionData' => $trans,
            ],
        ]);
    }

    /**
     * 根据交易哈希获取交易收据。
     */
    #[RequestMapping(path: 'getTransactionReceipt')]
    public function getTransactionReceipt(): ResponseInterface
    {
        // 交易哈希
        $transHash = $this->request->input('transHash');
        // 交易收据
        $result = $this->ethClient->eth_getTransactionReceipt($transHash);
        return $this->responseJson([
            'code' => 200,
            'msg' => 'Eth/getTransactionReceipt',
            'data' => $result,
        ]);
    }
}
```
