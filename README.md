import { motion } from 'framer-motion';
import { useState, useEffect } from 'react';
import { Card, CardContent } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import Link from 'next/link';
import { useTheme } from 'next-themes';
import Image from 'next/image';
import Head from 'next/head';
import { useRouter } from 'next/router';
import Script from 'next/script';
import { loadStripe } from '@stripe/stripe-js';
import { Input } from '@/components/ui/input';
import { Dialog, DialogContent, DialogTitle } from '@/components/ui/dialog';

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY);

export default function CreativeSpark() {
  const { theme, setTheme } = useTheme();
  const router = useRouter();
  const [promoCode, setPromoCode] = useState('');
  const [discount, setDiscount] = useState(0);
  const [showChat, setShowChat] = useState(false);

  useEffect(() => {
    window.gtag('config', 'G-XXXXXXXXXX', { page_path: router.pathname });
  }, [router.pathname]);
  
  const handleCheckout = async (service, price) => {
    const stripe = await stripePromise;
    const finalPrice = price - discount;
    const response = await fetch('/api/checkout', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ service, price: finalPrice }),
    });
    const session = await response.json();
    await stripe.redirectToCheckout({ sessionId: session.id });
  };

  const applyPromoCode = () => {
    if (promoCode === 'SPARK10') {
      setDiscount(10);
    } else {
      setDiscount(0);
    }
  };

  return (
    <>
      <Head>
        <title>CreativeSpark | Freelance Studio</title>
        <meta name="description" content="CreativeSpark offers professional video editing and graphic design services." />
        <meta name="keywords" content="video editing, graphic design, branding, social media content" />
        <meta name="author" content="CreativeSpark" />
      </Head>

      <Script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></Script>
      <Script>
        {`
          window.dataLayer = window.dataLayer || [];
          function gtag(){dataLayer.push(arguments);}
          gtag('js', new Date());
        `}
      </Script>
      
      <div className="min-h-screen bg-gray-100 dark:bg-gray-900 text-gray-900 dark:text-white">
        <header className="text-center py-10 flex justify-between px-5 items-center">
          <h1 className="text-4xl font-bold">CreativeSpark</h1>
          <motion.button whileHover={{ scale: 1.1 }} whileTap={{ scale: 0.9 }} onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')} className="p-2 bg-gray-300 dark:bg-gray-700 rounded-lg">
            {theme === 'dark' ? '☀️' : '🌙'}
          </motion.button>
        </header>
        
        {/* Services with Promo Code & Checkout */}
        <section className="py-10 px-5 bg-gray-200 dark:bg-gray-800">
          <h2 className="text-3xl font-semibold text-center mb-6">Services</h2>
          <Input className="w-full p-2 mb-4" placeholder="Enter Promo Code" value={promoCode} onChange={(e) => setPromoCode(e.target.value)} />
          <Button onClick={applyPromoCode} className="mb-6">Apply Code</Button>
          <motion.div className="grid grid-cols-1 md:grid-cols-2 gap-6" initial={{ opacity: 0 }} animate={{ opacity: 1 }} transition={{ duration: 0.8 }}>
            <Card className="p-4 bg-white dark:bg-gray-900 shadow-lg rounded-lg">
              <CardContent>
                <h3 className="text-xl font-medium">Video Editing</h3>
                <p className="text-gray-600 dark:text-gray-300">Professional video edits for YouTube, ads & social media.</p>
                <Button onClick={() => handleCheckout('Video Editing', 4999)} className="mt-4 bg-blue-500 text-white w-full py-2 rounded-lg hover:bg-blue-600">
                  Buy Now - ${(49.99 - discount).toFixed(2)}
                </Button>
              </CardContent>
            </Card>
            <Card className="p-4 bg-white dark:bg-gray-900 shadow-lg rounded-lg">
              <CardContent>
                <h3 className="text-xl font-medium">Graphic Design</h3>
                <p className="text-gray-600 dark:text-gray-300">Logos, branding, and social media graphics.</p>
                <Button onClick={() => handleCheckout('Graphic Design', 3999)} className="mt-4 bg-green-500 text-white w-full py-2 rounded-lg hover:bg-green-600">
                  Buy Now - ${(39.99 - discount).toFixed(2)}
                </Button>
              </CardContent>
            </Card>
          </motion.div>
        </section>

        {/* Live Chat Button */}
        <Button onClick={() => setShowChat(true)} className="fixed bottom-5 right-5 bg-purple-500 text-white p-4 rounded-full shadow-lg">
          💬 Chat
        </Button>

        {/* Live Chat Modal */}
        {showChat && (
          <Dialog open={showChat} onClose={() => setShowChat(false)}>
            <DialogContent>
              <DialogTitle>Live Chat</DialogTitle>
              <p className="text-gray-600 dark:text-gray-300">How can we assist you?</p>
              <Input className="w-full p-2 mt-2" placeholder="Type your message..." />
              <Button onClick={() => setShowChat(false)} className="mt-4">Send</Button>
            </DialogContent>
          </Dialog>
        )}
      </div>
    </>
  );
}
