# eth-client
php eth 交易客户端
# 在此库的基础上做修改
https://github.com/myxtype/ethereum-client
# 在 Hyperf-3(PHP8.1) 框架上使用
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
    /*
     * eth 客户端
     */
    protected EthClient $ethClient;

    public function __construct()
    {
        $client = new Client([
            // eth 节点
            'base_uri' => 'https://eth-goerli.g.alchemy.com/v2/cQ_wTHz6237vKR8yagHHTyrv1XPug_Oj',
            'timeout' => 10,
        ]);
        $this->ethClient = new EthClient($client);
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
        // gas 预估值
        $trans['gas'] = dechex((int) hexdec($this->ethClient->eth_estimateGas($trans)));
        // gas 手续费
        $trans['gasPrice'] = $this->ethClient->eth_gasPrice();
        $trans['nonce'] = $this->ethClient->eth_getTransactionCount($from, 'pending');

        // 发送交易请求，返回交易哈希
        $txid = $this->ethClient->sendTransaction($trans);
        // 交易收据
        $transactionReceipt = $this->ethClient->eth_getTransactionReceipt($txid);

        return $this->responseJson([
            'code' => 200,
            'msg' => 'Eth/Transaction',
            'data' => [
                'txid' => $txid,
                'transactionReceipt' => $transactionReceipt,
                'transactionData' => $trans,
            ],
        ]);
    }
}

```
