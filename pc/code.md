## 弹出扫码框

```
export default ({ dispatch, logined }: { dispatch: (action: Action) => void; logined: () => void }) => {
    const [expired, setExpired] = useState(false);
    const [qrinfo, setQrinfo] = useState('');
    const [failed, setFailed] = useState(false);
    const expiredTask = useRef<any>();

    const getQrcodeHandler = useCallback(
        (url: string) => {
            setExpired(false);
            setFailed(false);
            // 发现是 QQ 的连接
            setQrinfo(url);

            clearTimeout(expiredTask.current);
            expiredTask.current = setTimeout(() => {
                setExpired(true);
                dispatch({ type: LoginActions.QRCODE_EXPIRED, payload: {} });
            }, 600 * 1000);
        },
        [expired, dispatch]
    );

    const getQrcodeFailure = useCallback(() => {
        setExpired(true);
        setFailed(true);
        dispatch({ type: LoginActions.QRCODE_EXPIRED, payload: {} });
    }, [failed]);

    const createOption = useMemo(() => {
        return { qrcode: getQrcodeHandler, logined, failed: getQrcodeFailure };
    }, [getQrcodeHandler, getQrcodeFailure, logined]);

    useEffect(() => {
        if (qrinfo === '') {
            dispatch({ type: LoginActions.QRCODE_CREATE, payload: createOption });
        }

        return () => {
            clearTimeout(expiredTask.current);
            dispatch({ type: LoginActions.QRCODE_DISPOSE, payload: {} });
        };
    }, []);

    const reflush = useCallback(() => {
        setExpired(false);
        dispatch({ type: LoginActions.QRCODE_CREATE, payload: createOption });
    }, [expired, failed]);

    return (
        <div className={css.qrLogin}>
            <p>
                请打开
                <span>企鹅体育APP</span>
                <span>微信</span>
                <span>QQ</span>
                <br />
                扫描二维码登录
            </p>
            <div className={css.qrcode} onClick={reflush}>
                {qrinfo && qrinfo !== '' && <Qrcode url={qrinfo} />}
                {expired && (
                    <div className={css.expiredMask}>
                        {failed ? '二维码获取失败' : '二维码已失效'}
                        <div className={css.reflush} onClick={reflush}>
                            点击刷新
                        </div>
                    </div>
                )}
            </div>
        </div>
    );
};
```

### 二维码saga
```
// qrcode.tsx
// Qrcode.toDataURL(url, { margin: 0 }) 去转变成图像连接
import { useState } from 'react';
import Qrcode from 'qrcode';
import ImgLoader from './img-loader';

export default ({ url, className }: { className?: string; url: string }) => {
    const [QrUrl, update] = useState<string>('');
    Qrcode.toDataURL(url, { margin: 0 }).then(url => {
        update(url);
    });

    return <ImgLoader alt='' src={QrUrl} className={className} />;
};
```

```
// 可以看到是请求 url 返回 result，将 url 传入 qrcode，也就是 getQrcodeHandler。
// 调用 中的 setQrinfo(), 将 url 传入
export function* quickLogin() {
    while (true) {
        const {
            payload: { qrcode, logined, failed }
        } = yield take(LoginActionTypes.QRCODE_CREATE);
        const { result, dispose } = yield race({
            result: call(request, { url: apiUrl.LOGIN_QRCODE_CREATE }),
            dispose: take(LoginActionTypes.QRCODE_DISPOSE)
        });

        if (dispose) {
            //提前销毁
            continue;
        }

        if (yield call(vaildCode, result)) {
            if (result.url) {
                yield call(qrcode, result.url);
                const checkTask = yield fork(checkQrcode, result.qr_code, logined, failed);
                yield put({
                    type: LoginActionTypes.SET_ERCODE,
                    payload: {
                        code: result.qr_code
                    }
                });

                yield race({ expired: take(LoginActionTypes.QRCODE_EXPIRED), dispose: take(LoginActionTypes.QRCODE_DISPOSE) });

                yield call(cancelAll, [checkTask]);
            } else {
                //yield call(showError, result.message ?? '二维码地址错误');
                yield call(failed);
            }
        } else {
            //yield call(showError, result.message ?? '获取二维码失败');
            yield call(failed);
        }
    }
}
```